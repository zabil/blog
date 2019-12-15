---
title: Why we built Gauge.
description: The story behind a new kind of testing framework.
date: 2018-12-14
layout: layouts/post.njk
---

<link rel="canonical" href="https://blog.getgauge.io/why-we-built-gauge-6e31bb4848cd" />

At [ThoughtWorks](//thoughtworks.com) we place a high value on test automation. We’ve built a lot of testing tools. We [talk and write](https://www.thoughtworks.com/insights/software-testing) about how important testing is. And that’s because our teams rely on tests to quickly detect and fix problems in every stage of the build pipeline. But the truth is, it’s tough to maintain or debug tests, especially functional tests despite following the [test pyramid](https://martinfowler.com/bliki/TestPyramid.html).

## The importance of readable and reusable tests

From our experience, test suites need significant effort to stay readable and reusable. To explain this, let’s develop a simple test for automating the browser to google “testing frameworks” using Selenium.

```
void test() {
 // Invoke commands
 driver.get("google.com");
 // Use locators/xpath
 WebElement element = 
 	driver.findElement(By.name("q"));
 // Provide data
 element.sendKeys("testing frameworks");
 element.submit();
}
```
A test like this is verbose and not reusable by another test. That is until we move a few lines to a method, pass the search text as a parameter and call that method. For example

```
void google(String query) { 
 driver.get("google.com");
 WebElement element = 
 	driver.findElement(By.name("q"));
 element.sendKeys(query);
 element.submit();
}

void testOne() {
	google("automated testing");
}

void testTwo() {
	google("flying foxes");
}
```

This test script is easier to read and avoids repetition.

> Design made it better. But designing test cases are subjective to opinions and programming languages.

For example, [Selenium](https://www.seleniumhq.org/docs/06_test_design_considerations.jsp) recommends the Page Object design pattern. Let’s use it to refactor the same test.

```
class GoogleHomePage {
 GoogleHomePage(Webdriver driver) {
  driver.get("google.com");
  this.driver = driver;
 }
 void searchFor(String query) {
  WebElement element = 
  	driver.findElement(By.name("q"));
  element.sendKeys("testing frameworks");
  element.submit();
 }
}
void testOne() {
 GoogleHomePage page = new GoogleHomePage();
 page.searchFor("testing frameworks")
}
void testTwo() {
 GoogleHomePage page = new GoogleHomePage();
 page.searchFor("flying foxes")
}
```

This test is now reusable but not very readable because a programming language like Java adds syntax and OOP design noise like Classes, Objects, Methods, Variables etc. It is often hard to use these patterns. It breaks the flow of writing tests because one needs to design first and and then write the test case. It is also counterproductive adding eleven lines of code to reuse two lines.

Over time, teams over-engineer test suites with design opinions making it harder and harder to write new tests or maintain existing ones. We feel it’s important to minimize or eliminate this design process. Let’s try this on the original example by using a comment to describe the test.

```
// Google for testing frameworks
void test() {
 driver.get("google.com");
 driver.findElement(By.name("q"))
  .sendKeys("testing frameworks");
 driver.findElement(By.name("q")).submit();
}
```

This reads better but it’s not reusable. What if we parse the comment for parameters and pass it to the test?

```
// Google for <query>
void test(String query) {
 driver.get("google.com");
 driver.findElement(By.name("q")).sendKeys(query);
 driver.findElement(By.name("q")).submit();
}
```

Now let’s write the comments down in a file and specify the parameters.

```
Google for "testing frameworks"
Google for "flying foxes"
```

A parser can read this file, match it to the comment on the method, pass parameters and run it. The test is now both readable and reusable without the design overhead and with lesser code.

## Introducing Gauge

Starting with these ideas we built Gauge — a free and open source test automation framework. Gauge makes test automation a natural part of the software development cycle by removing any hurdle that comes in the way of writing and maintaining acceptance tests. We believe that Gauge is:

1. Easy to setup
2. Easy to learn
3. Easy to maintain
4. Easy to extend

## Easy to setup

### Single binary

There are no prerequisites for installing and running Gauge. It’s a single binary available for all platforms. Installing Gauge is as simple as unzipping it to a folder.

```
$ unzip -o gauge.zip -d /usr/local/bin
```

### Templates

Instead of cobbling jars/modules/gems/pips, creating directory layouts, writing build scripts before even getting to write tests, simply run

```
$ gauge init js # or csharp, java, python, ruby
```

to set up a sample JavaScript test project. To help you quick-start projects we’ve set up templates for all popular languages and tools. Teams can also create and share templates to bootstrap custom projects.

### Runner

Unlike production code, tests are not packaged or deployed. They are checked out from a version control system and run locally or on a CI/CD server. We’ve designed Gauge’s runner to work with your test’s source code.

```
$ gauge init java  # or csharp, js, python, ruby
$ gauge run specs
```

Notice how Gauge compiles the java test sources before running them. There’s no need to setup custom build scripts to run Gauge specifications.

## Easy to learn

### Simple Syntax

Gauge specifications are written in Markdown format. Markdown is a popular easy-to-write and easy-to-read plain text format. Gauge inherits and extends Markdown’s syntax to describe specifications, for example(1)

```
# Greeter
## Greet the world
* Say hello to the "World"
```

The pound or hash symbols (# and ##), markdown syntax for headers, are used by Gauge for specification and scenario descriptions. The asterisk (*), which is markdown syntax for list items, is used to describe steps in a scenario.

Writing a test specification is as easy as writing down the testing flow. Unlike BDD tools, Gauge does not prescribe the process with a strict syntax.

You can use any Markdown parser to parse or your favorite Markdown editor (and its features) to write and preview Gauge specifications.

### Simple implementations

After writing a specification, you can implement the step using a programming language. An implementation for example(1) in JavaScript looks like example(2)

```
step("Say hello to the <person>", (person) => {
 console.log(`Hello ${person}!`)
})
```

Example(1) along with example(2) is an executable Gauge specification. Notice that the step text also works like a comment describing the step. You can save the specification as `specs/greeter.md`, implementation as `tests/greeter.js` and run it using Gauge to see the results.

### Familiarity

Gauge speaks your business language and supports internationalization by default.

Whether it’s English

```
# Find movies playing near me
## Search and confirm movie
* Search for theatres playing "Star wars" in "Bangalore"
* Check if "IMAX" is playing star wars at "7:30PM"
```

or German

```
# Suche Filme, die in meiner Nähe gespielt werden
## Suche und bestätige Filme
* Suche nach Kinos, die "Star wars" in "Bangalore" spielen
* Überprüfe, ob "IMAX" star wars um "19:30" spielt
```

and with official support for C#, Java, JavaScript, Python and Ruby, you can write tests in a language you are familiar with. Teams can leverage existing libraries, tools (Selenium, Appium et al) and IDE’s (Visual Studio, Intellij IDEA, Visual Studio Code) to quickly start their testing projects and make it a part of their project’s ecosystem.

## Easy to maintain

### Re-usability

Gauge specifications are designed for re-usability. While writing your specifications you can

* Reuse steps in specifications across scenarios
* Defines steps to run before every scenario
* Combine steps into concepts
* Define aliases for steps performing the same action

and implement them after you are done. This reduces a lot of boilerplate code by not making use of a programming language for re-usability. You can piece together steps along with their implementations, much like blocks, for future tests.

To understand this better let’s write a Gauge specification for testing a website for booking movies. We start with a specification for searching blockbuster movies playing at a location. Example(3)

```
# Search for movies
## Search for blockbusters
* Search for theaters playing "Avengers" in "Bangalore"
```

We can reuse this step in another scenario, let’s say searching for Oscar winning movies.

```
# Search for movies
## Search for blockbusters
* Search for theaters playing "Avengers" in "Bangalore"
## Search 2017 Oscar winners
* Search for theaters playing "The shape of water" in 
  "Bangalore"
```

Notice that, by changing a few parameters, we’ve re-used the same step in example(3) to search for a different movie. If you don’t want to repeat setting the location in every scenario split the step and set the location before every scenario.

```
# Search for movies
* Set location as "Bangalore"
## Search for blockbusters
* Search for theaters playing "Avengers"
## Search 2017 Oscar winners
* Search for theaters playing "The shape of water"
```

You can combine steps into a concept to avoid repeating a sequence of steps or improving readability for example, if booking a ticket involves a series of actions, define it as a concept

```
# Book <number> tickets
* Pick <number> seats
* Login in as "John"
* Pay using credit card
* Check ticket
```

and reuse it in your specification just like any other step with a single line

```
# Search for movies
* Set location as "Bangalore"
## Search for blockbusters
* Search for theaters playing "Avengers"
* Book "2" tickets
## Search 2017 Oscar winners
* Search for theaters playing "The shape of water"
```

You can even use a concept inside another concept!

```
# <number> tickets for <time> show please!
* Pick the <time> show
* Book <number> tickets
```

```
# Search for movies
* Set location as "Bangalore"
## Search for blockbusters
* Search for theaters playing "Avengers"
* Book "2" tickets
## Search 2017 Oscar winners
* "2" tickets for "7:30PM" show please!
```

As we’ve mentioned earlier. Organizing your specifications first with steps and concepts greatly reduces duplicate code and repetition of your programming logic as step implementations are executed wherever they are reused.

### Refactoring

With in-built support for refactoring, Gauge handles changes to steps and concepts seamlessly. For example changing

```
* "2" tickets for "7:30PM" show please!
```

to

```
* "2" tickets for "7:30PM" show and order "popcorn"!
```

modifies scenarios, concepts and the step implementation where it’s used. This allows you to frequently refine tests when requirements change without breaking anything.

## Easy to extend

### Plugins

Gauge is modular by design and extensible using plugins. You can write plugins for Gauge in any language. Features like the runner, reports, screenshots, etc. are plugins bundled with Gauge. This allows Gauge to

* Support a new language
* Write custom reports
* Publish test results to a test management system
* Generate documentation

This makes building other testing frameworks or even products using Gauge easy.

### Tools

Gauge also runs as a Language Server and uses the protocol to add powerful editing features for Gauge specifications like

* Auto completion
* Step and implementation references.
* Refactoring
* Running specifications and scenarios (within the IDE)
* Locating a the step’s implementation
* Listing all scenarios and quickly jump to it (like symbols in a programming * language)
* Displaying errors and warnings while writing specifications
* Formatting

for a growing list of IDE’s that support Language Server Protocol like Visual Studio Code, Sublime, Atom, Eclipse, Vim, Emacs or even edit in a browser.

## Is Gauge a BDD tool?

No. Gauge is built for testing. BDD tools like Cucumber, on the other hand, are built primarily for collaboration and not testing. Gauge will always focus on solving problems with test authoring and maintenance.

## Looking forward

We know Gauge solves real problems with test maintenance because we have a growing community of early adopters who took the risk of using a beta product on their build pipelines. They’ve given us valuable feedback, pull requests, encouragement and have recommended Gauge to their colleagues.

With the 1.0 stable out we’d like to thank them for supporting our ideas and approach. We hope you’ll find it as useful as we do for writing clean and maintainable acceptance tests!