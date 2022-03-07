# gocuke

`gocuke` is a Gherkin-based BDD testing library for golang. It is inspired by
[godog](https://github.com/cucumber/godog) and [gobdd](https://github.com/go-bdd/gobdd).

## Why another golang BDD library?

Sorry guys. I tried godog and gobdd and wanted a specific developer UX that
I couldn't achieve with either. godog was not a good fit for the same reasons
as that gobdd was created (specifically tight integration with `*testing.T`).
Looking at the source code for gobdd, it wasn't that complex but needed to
be updated to a new versions of cucumber/gherkin-go and cucumber/messages-go
which was basically a complete rewrite anyway.

This library offers:
* tight integration with `*testing.T` (use any standard assertion library or mocking framework)
* support for passing context between steps using suites which offers better 
  type safety than other generic context approaches
* auto-discovery of step definitions defined as test suite methods and step
  definition suggestions for minimal configuration
* property-based testing via https://github.com/flyingmutant/rapid
* user-friendly wrapper for data tables
* support for big integers and big decimals (via https://github.com/cockroachdb/apd)
* parallel test execution by default
* full support for all of the latest Gherkin features including rules (via
  the latest cucumber/gherkin-go and cucumber/messages-go)

## Usage

### Step 1: Define some Gherkin

In a file `features/simple.feature`:

```gherkin
Feature: simple

Scenario: eat cukes
  Given I have 5 cukes
  When I eat 3
  Then I have 2 left
```

### Step 2: 

Setup the test suite:

In a file simple_test.go:

```go
package simple

import (
	"gocuke"
	"testing"
)

func TestMinimal(t *testing.T) {
	gocuke.NewRunner(t, func(t gocuke.TestingT) gocuke.Suite {
		return &suite{TestingT: t}
	}).Run()
}

type suite struct {
	gocuke.TestingT
}
```

When running tests you should get these suggestions:
```go
        func (s *suite) IEat(a int64) {
            panic("TODO")
        }
        
        func (s *suite) IHaveLeft(a int64) {
            panic("TODO")
        }
        
        func (s *suite) IHaveCukes(a int64) {
            panic("TODO")
        }
```

Copy these definitions into `simple_test.go`

### Step 3: Implement Step Definitions

```go
type suite struct {
	gocuke.TestingT
	cukes int64
}

func (s *suite) IHaveCukes(a int64) {
	s.cukes = a
}

func (s *suite) IEat(a int64) {
	s.cukes -= a
}

func (s *suite) IHaveLeft(a int64) {
	if a != s.cukes {
		s.Fatalf("expected %d cukes, have %d", a, s.cukes)
	}
}
```

Your tests should now pass!