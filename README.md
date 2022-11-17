# About `cloudsec`
`cloudsec` is a library and toolkit for formally analyzing security policies in cloud/API systems using Python. It has a modular and extensible architecture allowing it to support multiple backends; 
initially, [z3](https://github.com/z3prover) and [cvc5](https://cvc5.github.io/) will be supported. 
The `cloudsec` project takes influence from a number of related projects, such as the [Z3 Firewall Checker](https://github.com/Z3Prover/FirewallChecker).


## Background
Cloud/API systems are commonly secured through the use of security *policies* -- rules governing the set of  actions that agents (users, services, etc) are authorized to take within the system. For example, AWS uses IAM Policies; Kubernetes provides pod security policies, network policies, RBAC, etc. 

Within this context, a common question arises: given two sets of security policies, `P` and `Q`, are the rules generated by `P` and `Q` equivalent, or is one set strictly more permissive than the other? A particular common case is to let `P` be some existing policy set and let `Q` be a policy set that includes a security vulnerability. In this case, determining that `P` is equivalent to `Q` (or even just that `P=>Q`) establishes that the existing policy set contains a vulnerability. These are the types of questions `cloudsec` tries to help answer.


## Trying it Out

Docker images can be built to try out the `cloudsec` software. The `Makefile` can be used to generate the
images:

```
# generate the images --
$ make build
```

With the images generated, start a container with the `cloudsec` software and examples using `docker`:.
Have a look at the `examples_z3.py` file, contained within the `examples` directory, for all the definitions
of the objects used below. 

```
$ docker run -it --rm --entrypoint=bash --name=sec  jstubbs/cloudsec-exs
```

Start a Python shell and import the examples:

```
# from within the container started above,

>>> from examples_z3 import *

>>> result = checker.p_implies_q()

# p => q is True, meaning that the p policy set is less permissive than the q policy set
# i.e., any activity allowed by p is also allowed by q.
>>>  result.proved
True

>>> result = checker.q_implies_p()

# q => p is False, however, because q is NOT less permissive than p; that is, q allows some activities
# that p does not allow.
>>> result.proved
False

# In fact, cloudsec was able to find a counter example to the statement q => p; i.e., it was able to find
# an activity allowed by q but not by p.
>>> result.found_counter_ex
True

# the result.model contains a counter example to the q => p statement; i.e., it contains an example of
# an activity that is allowed by the q policy set but not by the q policy set. 
>>> result.model
[action = "PUT",
 resource_path = "s2/home/jstubbs",
 resource_service = "files",
 resource_tenant = "a2cps",
 principal_username = "jstubbs",
 principal_tenant = "a2cps"]

```

## Development

`cloudsec` includes a test suite based on `pytest`. The Makefile can be used to build
the tests container image and run the tests:

```
# Build the tests image 
$ make build-tests
```

```
# Run the tests
$ make test
```