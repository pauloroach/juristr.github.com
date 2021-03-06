---
layout: post
title: "An Example of Test-Driven Bugfixing"
date: 2012-07-19
comments: true
tags: [  JavaScript, testing ]
---

Unfortunately many people still don't use a test-driven approach during development. That's sad given the bunch of <a href="http://blog.js-development.com/search/label/Unit%20Testing" target="_blank">advantages automated tests give you</a>. One of my favorite is regression testing. Since I just spotted a bug in my codebase, I'll take the opportunity to quickly document the bugfixing approach I'm usually applying.

The bug is the one I described in my [previous post](/blog/2012/07/why-extendsomeobj-anotherobj-might-be/). Please read it to be able to properly follow this one here.

<h2>Intro</h2>
<span style="background-color: white;">
  I basically have the following function defined in my ajax.js file of our
</span>
<a href="http://javascriptmvc.com/" style="background-color: white;" target="_blank">JavaScriptMVC</a>
<span style="background-color: white;">extension library.</span>
<br />
<pre class="brush:javascript">$.Model.findAllPaged = function (params, currentPage, pageSize, success, error) {<br />    return $.ajax({<br />        url: this.resolveUrl(this.baseUrl || this.shortName),<br />        type: "GET",<br />        dataType: "json " + resolveClassNameForConverter(this.shortName) + ".models", //"account.models" for autowiring<br />        data: $.extend(params, { currentPage: currentPage, pageSize: pageSize }),<br />        success: success,<br />        error: error,<br />        fixture: "-restFindAll" // uses $.fixture.restDestroy for response.<br />    });<br />};</pre>
This function wraps a call with paging support to our server API. It can be invoked as follows:
<br />
<pre class="brush:javascript">Test.PersonAccount.findAllPaged(<br />   {}, <br />   currentPage, <br />   numPages,<br />   function (resultData) { ... },<br />   function (error) { ... });<br /></pre>
Pretty self-explanatory.
<br />
<h2>Create a failing test</h2>
The bug is basically in <b>line 6</b>
of the
<code>findAllPaged</code>
function as mentioned in my
<a href="http://blog.js-development.com/2012/07/why-extendsomeobj-anotherobj-might-be.html" target="_blank">previous post</a>
. So the first step <b>before even going to touch the buggy code</b>
is to create a
<b>failing</b>
test. This is extremely important as it
<br />
<ul>
  <li>
    <b style="background-color: white;">proves</b>
    <span style="background-color: white;">the presence of the bug</span>
  </li>
  <li>
    <span style="background-color: white;">gives a positive feedback once the bug has been resolved</span>
  </li>
  <li>
    <span style="background-color: white;">
      functions as a regression test, preventing the introduction of the bug in the future by someone else
    </span>
  </li>
</ul>
For the test I'm using
<a href="http://qunitjs.com/" target="_blank">QUnit</a>
, written by the jQuery team and already included by default in JavaScriptMVC. The test looks as follows:
<br />

<pre class="linenums">test("Calling Model.findAllPaged should not modify the passed parameter object", 1, function () {<br />    //arrange<br />    $.fixture["-restFindAll"] = function (settings, cbType) {<br />        var result = {<br />            page: 3,<br />            total: 7,<br />            records: 200,<br />            rows: [<br />                    {<br />                        id: 5,<br />                        firstname: "juri"<br />                    },<br />                    {<br />                        id: 6,<br />                        firstname: "juri123"<br />                    }<br />                 ]<br />        };<br /><br />        return [result];<br />    };<br /><br />    //act<br />    stop(500);<br />    var param = { currentPage: 0 };<br />    Test.PersonAccount.findAllPaged(param, 1, 10,<br />        function (data) {<br />            equals(param.currentPage, 0);<br />            start();<br />        },<br />        function (error) {<br />            ok(false, "Error: " + error.responseText);<br />            start();<br />        }<br />    );<br />});<br /></pre>
Some explanation to the test's anatomy:
<br />
<ul>
  <li>
    <b>line 1 -</b>
    this is the header of the test case specifying a test name in the form of a written string that should explain the purpose of the test. The "1" between the actual test function and the test name specifies the number of expected asserts. This is useful as JavaScript involves async actions and callbacks. So we want to make sure our test fails if no callbacks are being called for instance.
  </li>
  <li>
    <b>line 3 -</b>
    this line defines a fixture that substitutes the actual ajax call that is being made by the function under test. This is a build-in feature provided by the JavaScriptMVC framework.
    <a href="http://blog.js-development.com/2011/09/testing-javascript-mocking-jquery-ajax.html" target="_blank">It could be done manually as well</a>
    . This is important as we don't want to actually contact the server during the execution of the test.
  </li>
  <li>
    <b>line 24 -</b>
    stop(timeout-milliseconds) instructs the test-case to halt and only continue its execution when start() is invoked again.
    <a href="http://qunitjs.com/cookbook/#asynchronous_callbacks" target="_blank">Read more here</a>
    .
  </li>
  <li>
    <b>line 26 -</b>
    the actual&nbsp;invocation&nbsp;of the function to be tested.
  </li>
  <li>
    <b>line 28</b>
    - the assert
  </li>
</ul>
<div>Run the test...</div>

![](/blog/assets/imgs/qunit_failing_test.png)

<div>...it fails. Perfect.</div>
<div>
  <br />
</div>
<h2>Now, go and fix the bug</h2>
<div>
  To fix the bug I substitute the buggy line as already explained in my
  <a href="http://blog.js-development.com/2012/07/why-extendsomeobj-anotherobj-might-be.html" target="_blank">previous post</a>
  . Basically, exchanging
  <b>line 6</b>
  of the
  <code>findAllPaged</code>
  function with the following:
</div>
<pre class="brush:javascript">...<br />data: $.extend({}, params, { currentPage: currentPage, pageSize: pageSize }),<br />...<br /></pre>
<h2>
  <span style="background-color: white;">Re-run the test</span>
</h2>
<div>
  <span style="background-color: white;">Re-running the test...</span>
</div>

![](/blog/assets/imgs/qunit_succeeding_test.png)

<div>
  <span style="background-color: white;">
    ..
    <b>success</b>
    . The bug is fixed!
  </span>
</div>
Refactor (if necessary) and re-run
<b>all the tests</b>
again to see if your modification didn't break any other tests.
<br />
<br />
That's it!