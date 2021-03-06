[![Build Status](https://travis-ci.org/DATA-DOG/godog.svg?branch=master)](https://travis-ci.org/DATA-DOG/godog)
[![GoDoc](https://godoc.org/github.com/DATA-DOG/godog?status.svg)](https://godoc.org/github.com/DATA-DOG/godog)

# Godog

<p align="center"><img src="/logo.png" alt="Godog logo" style="width:250px;" /></p>

**The API is likely to change a few times before we reach 1.0.0**

Package godog is the official Cucumber BDD framework for Golang, it merges
specification and test documentation into one cohesive whole. The author
is a core member of [cucumber team](https://github.com/cucumber).

What is behavior-driven development, you ask? It’s the idea that you start
by writing human-readable sentences that describe a feature of your
application and how it should work, and only then implement this behavior
in software.

The project is inspired by [behat][behat] and [cucumber][cucumber] and is
based on cucumber [gherkin3 parser][gherkin].

**Godog** does not intervene with the standard **go test** command and its
behavior. You can leverage both frameworks to functionally test your
application while maintaining all test related source code in **_test.go**
files.

**Godog** acts similar compared to **go test** command. It uses go
compiler and linker tool in order to produce test executable. Godog
contexts needs to be exported same as Test functions for go test.

**Godog** ships gherkin parser dependency as a subpackage. This will
ensure that it is always compatible with the installed version of godog.
So in general there are no vendor dependencies needed for installation.

The following about section was taken from
[cucumber](https://cucumber.io/) homepage.

## About

#### A single source of truth

Cucumber merges specification and test documentation into one cohesive whole.

#### Living documentation

Because they're automatically tested by Cucumber, your specifications are
always bang up-to-date.

#### Focus on the customer

Business and IT don't always understand each other. Cucumber's executable
specifications encourage closer collaboration, helping teams keep the
business goal in mind at all times.

#### Less rework

When automated testing is this much fun, teams can easily protect
themselves from costly regressions.

### Install

    go get github.com/DATA-DOG/godog/cmd/godog

### Example

The following example can be [found
here](/examples/godogs).

#### Step 1

Given we create a new go package **$GOPATH/src/godogs**. From now on, this
is our work directory `cd $GOPATH/src/godogs`.

Imagine we have a **godog cart** to serve godogs for lunch. First of all,
we describe our feature in plain text - `vim
$GOPATH/src/godogs/features/godogs.feature`:

``` gherkin
# file: $GOPATH/src/godogs/features/godogs.feature
Feature: eat godogs
  In order to be happy
  As a hungry gopher
  I need to be able to eat godogs

  Scenario: Eat 5 out of 12
    Given there are 12 godogs
    When I eat 5
    Then there should be 7 remaining
```

As a developer, your work is done as soon as you’ve made the program behave as
described in the Scenario.

**NOTE:** same as **go test** godog respects package level isolation. All
your step definitions should be in your tested package root directory. In
this case - `$GOPATH/src/godogs`

#### Step 2

If godog is installed in your GOPATH. We can run `godog` inside the
**$GOPATH/src/godogs** directory. You should see that the steps are
undefined:

![Undefined step snippets](/screenshots/undefined.png?raw=true)

If we wish to vendor godog dependency, we can do it as usual, using tools
you prefer:

    git clone https://github.com/DATA-DOG/godog.git $GOPATH/src/godogs/vendor/github.com/DATA-DOG/godog

It gives you undefined step snippets to implement in your test context.
You may copy these snippets into your `godogs_test.go` file.

Our directory structure should now look like:

![Directory layout](/screenshots/dir-tree.png?raw=true)

If you copy the snippets into our test file and run godog again. We should
see the step definition is now pending:

![Pending step definition](/screenshots/pending.png?raw=true)

You may change **ErrPending** to **nil** and the scenario will
pass successfully.

Since we need a working implementation, we may start by implementing only what is necessary.

#### Step 3

We only need a number of **godogs** for now. Lets keep it simple.

``` go
/* file: $GOPATH/src/godogs/godogs.go */
package main

// Godogs available to eat
var Godogs int

func main() { /* usual main func */ }
```

#### Step 4

Now lets implement our step definitions, which we can copy from generated
console output snippets in order to test our feature requirements:

``` go
/* file: $GOPATH/src/godogs/godogs_test.go */
package main

import (
	"fmt"

	"github.com/DATA-DOG/godog"
)

func thereAreGodogs(available int) error {
	Godogs = available
	return nil
}

func iEat(num int) error {
	if Godogs < num {
		return fmt.Errorf("you cannot eat %d godogs, there are %d available", num, Godogs)
	}
	Godogs -= num
	return nil
}

func thereShouldBeRemaining(remaining int) error {
	if Godogs != remaining {
		return fmt.Errorf("expected %d godogs to be remaining, but there is %d", remaining, Godogs)
	}
	return nil
}

func FeatureContext(s *godog.Suite) {
	s.Step(`^there are (\d+) godogs$`, thereAreGodogs)
	s.Step(`^I eat (\d+)$`, iEat)
	s.Step(`^there should be (\d+) remaining$`, thereShouldBeRemaining)

	s.BeforeScenario(func(interface{}) {
		Godogs = 0 // clean the state before every scenario
	})
}
```

Now when you run the `godog` again, you should see:

![Passed suite](/screenshots/passed.png?raw=true)

**Note:** we have hooked to **BeforeScenario** event in order to reset state. You may hook into
more events, like **AfterStep** to test against an error and print more details about the error
or state before failure. Or **BeforeSuite** to prepare a database.

#### Running Godog with go test

There was a question asked whether it is possible to run **godog** from
**go test** command. And the answer is yes. You can run it using go
[TestMain](https://golang.org/pkg/testing/#hdr-Main) func available since
go 1.4. In this case it is not necessary to have **godog** command
installed. See the following example:

``` go
/* file: $GOPATH/src/godogs/godogs_test.go */
package main

import (
	"flag"
	"os"
	"testing"

	"github.com/DATA-DOG/godog"
)

func TestMain(m *testing.M) {
	args := os.Args
	// args for godog
	os.Args = []string{
		args[0],
		"-f", "progress",
		"features",
	}

	status := godog.Run(func(s *godog.Suite) {
		FeatureContext(s)
	})

	os.Args = args
	flag.Parse()

	if st := m.Run(); st > status {
		status = st
	}
	os.Exit(status)
}
```

### References and Tutorials

- [how to use godog by semaphoreci](https://semaphoreci.com/community/tutorials/how-to-use-godog-for-behavior-driven-development-in-go)

### Documentation

See [godoc][godoc] for general API details.
See **.travis.yml** for supported **go** versions.
See `godog -h` for general command options.

See implementation examples:

- [rest API server](/examples/api)
- [rest API with Database](/examples/db)
- [godogs](/examples/godogs)

### FAQ

**Q:** Where can I configure common options globally?
**A:** You can't. Alias your common or project based commands: `alias godog-wip="godog --format=progress --tags=@wip"`

### Contributions

Feel free to open a pull request. Note, if you wish to contribute an extension to public (exported methods or types) -
please open an issue before to discuss whether these changes can be accepted. All backward incompatible changes are
and will be treated cautiously.

### License

All package dependencies are **MIT** or **BSD** licensed.

**Godog** is licensed under the [three clause BSD license][license]

[godoc]: http://godoc.org/github.com/DATA-DOG/godog "Documentation on godoc"
[golang]: https://golang.org/  "GO programming language"
[behat]: http://docs.behat.org/ "Behavior driven development framework for PHP"
[cucumber]: https://cucumber.io/ "Behavior driven development framework for Ruby"
[gherkin]: https://github.com/cucumber/gherkin-go "Gherkin3 parser for GO"
[license]: http://en.wikipedia.org/wiki/BSD_licenses "The three clause BSD license"
