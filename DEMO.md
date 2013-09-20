﻿<a name="title" />
# Building a SPA interface using Ember.js #

---
<a name="Overview" />
## Overview ##
This demo demonstrates a Windows Store frond end application (developed with HTML & JS) that allows users to take the quiz. It tours through already existing code that retrieves data from the REST API and shows the quiz running in a Windows Store application.

<a id="goals" />
### Goals ###
In this demo, you will see how to:

1. Retrieve data from a REST API from a Windows Store application.
1. Leverage WinJS data-binding capabilities to automatically update the UI.

<a name="technologies" />
### Key Technologies ###

- [HTML](http://www.w3schools.com/html/)
- [WinJS](http://msdn.microsoft.com/en-us/library/windows/apps/br229773.aspx)


<a name="setup" />
### Setup and Configuration ###
Follow these steps to setup your environment for the demo.

1. Open Visual Studio 2013.
1. Open the **GeekQuiz.sln** solution located under **source\end**.
1. Collapse the **GeekQuiz.Web** project node in **Solution Explorer**.
1. In Visual Studio, close all open files.

<a name="Demo" />
## Demo ##
This demo is composed of the following segments:

1. [Walkthrough of default.js](#segment1)
1. [Walkthrough of viewModel.js](#segment2)

<a name="segment1" />
### Walkthrough of default.js ###

1. Open the file **default.js** located inside the **js** folder of the **GeekQuiz** project.

1. Select the code highlighted in the following snippet.
	
	<!-- mark:12-17 -->
	````JavaScript
	app.onactivated = function (args) {
			  if (args.detail.kind === activation.ActivationKind.launch) {
					if (args.detail.previousExecutionState !== activation.ApplicationExecutionState.terminated) {
						 // TODO: This application has been newly launched. Initialize
						 // your application here.
					} else {
						 // TODO: This application has been reactivated from suspension.
						 // Restore application state here.
					}
					args.setPromise(WinJS.UI.processAll());

					var root = document.getElementById("root");
					var questionDiv = document.getElementById("question");
					var nextButton = document.getElementById("next");
					var controller = new QuestionController(root, questionDiv, nextButton);

					controller.nextQuestion();
			  }
		 };
	````

	> **Speaking point:** Explain that we our logic is going to be in a controller object and that the UI elements are going to be updated through data-binding.  We are basically creating the controller and asking for the next questions immediatly.

1. Highlight the following line (line #5 in the file):

	````JavaScript
	WinJS.Binding.optimizeBindingReferences = true;
	````

	> **Speaking Point:** Explain that this is required for every Windows Store application that takes advantage of data binding. Source [here](http://msdn.microsoft.com/en-us/library/windows/apps/jj215606.aspx).


<a name="segment2" />
### Walkthrough of viewModel.js ###

1. Open the file **viewModel.js** located inside the **js** folder of the **GeekQuiz** project.

1. Highlight the call to the `return WinJS.Class.define` function.

	> **Speaking point:** Explain that this the way that you can create classes with WinJS. Source [here](http://msdn.microsoft.com/en-us/library/windows/apps/br229813.aspx). The class that we are creating is a view model. The set of properties of a view model instance at a particular point in time represent the state of the view.

1. Highlight the line highlighted in the following code snippet:
	
	<!-- mark:6 -->
	````JavaScript
	return WinJS.Class.define(
        function (root, questionDiv, nextButton) {
            var self = this;

            var i;
            this.apiUrl = "http://localhost:50505/api/trivia";

            this.buttons = questionDiv.getElementsByTagName("button");

            this.question = {
                title: "Empty",
                id: 0,
                option1: {},
                option2: {},
                option3: {},
                option4: {},
                correct: false,
            };
	````

	> **Speaking point:** Explain that this is the end point of the REST API.

1. Highlight the lines highlighted in the following code snippet:
	
	<!-- mark:8-18 -->
	````JavaScript
	return WinJS.Class.define(
        function (root, questionDiv, nextButton) {
            var self = this;

            var i;
            this.apiUrl = "http://localhost:50505/api/trivia";

            this.buttons = questionDiv.getElementsByTagName("button");

            this.question = {
                title: "Empty",
                id: 0,
                option1: {},
                option2: {},
                option3: {},
                option4: {},
                correct: false,
            };
	````

	> **Speaking point:** Explain that we are retrieving all the buttons that will be the question options. We are also creating a question with default values. These values will be updated when we retrieve each question from the REST API, and will be automatically updated through data-binding.

1. Highlight the code included in the following snippet:

	````JavaScript
	this.state = this.states.loading;

	this.eventListeners = [];

	for (i = 0; i <= 3; i++) {
		 this.eventListeners[i] = function (num) {
			  return function () {
					var j;
					// we are always the same buttons, need to clear event listeners
					for (j = 0; j < self.buttons.length; j++) {
						 self.buttons[j].removeEventListener("click", self.eventListeners[i]);
					}
					self.sendAnswer(self.question, self.question["option" + num]);
			  };
		 }(i + 1);
	}

	nextButton.addEventListener("click", function () {
		 self.nextQuestion.apply(self, arguments);
	});
	````

	> **Speaking point:** Explain that we are using `states` as an enumerator of the possible view states. We are also creating functions for the event listeners.
	All event listeners need to be removed once an option is clicked, as we are re-using the buttons for all questions.

1. Highlight the code included in the following snippet:

	````JavaScript
	WinJS.Binding.processAll(root, this);

	this.observable = WinJS.Binding.as(this);
	````

	> **Speaking point:** We are setting the view model instance as the binding source for the `root` element. After that, we are creating an observable (proxy) for the view model, so changes to properties are automatically reflected in the UI.

1. Highlight the `nextQuestion` method, which is included in the following code snippet:

	````JavaScript
	nextQuestion: function () {
		 var self = this;

		 WinJS.xhr({
			  url: this.apiUrl
		 }).then(
			  function (response) {
					var j, q = JSON.parse(response.responseText);
					self.observable.question.id = q.id;
					self.observable.question.title = q.title;
					self.observable.question.option1 = q.options[0];
					self.observable.question.option2 = q.options[1];
					self.observable.question.option3 = q.options[2];
					self.observable.question.option4 = q.options[3];

					for (j = 0; j < self.buttons.length; j++) {
						 self.buttons[j].addEventListener("click", self.eventListeners[j]);
					}

					self.observable.state = self.states.showingQuestion;
			  }, function (error) {
					console.log(error);
			  });
	}
	````

	> **Speaking point:** Explain that it performs the web request and once it completes successfully the properties in the observable are updated, which automatically updates the UI. The `xhr` function returns a promise (conceptually similar to a `Task` in .NET) so it can be returned or chained with other promises. Explain that we are using WinJS facilities, but we could use any library (such as JQuery) for the AJAX calls.

1. Highlight the `sendAnswer` method, which is included in the following code snippet:

	````JavaScript
	sendAnswer: function(question, option) {
		 this.observable.state = this.states.loading;
		 var self = this;
		 console.log("web request");
		 WinJS.xhr({
			  url: self.apiUrl,
			  type: "post",
			  headers: { "Content-type": "application/json" },
			  data: JSON.stringify({ "questionId": question.id, "optionId": option.id })
		 }).then(function (response) {
			  var r = JSON.parse(response.responseText);
			  self.observable.question.correct = r;
			  self.observable.state = self.states.showingAnswer;
		 }, function (error) {
			  console.log(error);
		 })
	}
	````

	> **Speaking point:** Explain that this is similar to the `nextQuestion` method, but the main difference is that we are using an HTTP POST instead of a GET.

<a name="segment3" />
### Walkthrough of default.html ###