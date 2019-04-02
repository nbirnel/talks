A refactoring story
-------------------

refactoring a bit of terraform, moving an ec2 instance and it's related
security groups, DNS records, etc into a module.

After a few hours of work, `terraform plan`.

Whoops, I have another security group that still refers to the old
instance, not it's modularized version. Fix.

`terraform plan`. Looks good. `terraform apply`.

ssh in, after nuking the old instance from my known_hosts.

Take a look at `/var/log/cloud-init-output.log`

Oops, it can't get updates. I failed to pass the updates-egress security
group into the new module.

Disable termination protection. Terminate. Wait. `terraform plan`,
`terraform apply`. Wait. `ssh-keygen -R`, `ssh`,
`tail /var/log/cloud-init-output.log`.

Saltstack moved their repos around a bit since I last built this instance
two years ago.

Disable termination protection. Terminate. Wait. `terraform plan`,
`terraform apply`. Wait. `ssh-keygen -R`, `ssh`,
`tail /var/log/cloud-init-output.log`.

Oops, I didn't create the directory for the the nfs mount point.

Disable termination protection. Terminate. Wait. `terraform plan`,
`terraform apply`. Wait. `ssh-keygen -R`, `ssh`,
`tail /var/log/cloud-init-output.log`.

It is spelled ``mkdir``, not ``mkdri``.

Disable termination protection. Terminate. Wait. `terraform plan`,
`terraform apply`. Wait. `ssh-keygen -R`, `ssh`,
`tail /var/log/cloud-init-output.log`.

Some Types of SD Tests, and how they translate to IaC/DevOps tests
------------------------------------------------------------------

* unit tests (functional) -> unit tests, 
  but not functional: we are testing side-effects
* behavioral tests -> where are these going to live?
* integration tests -> This, certainly, is the same
* CI -> Yeeessss, but - we can CI code, or even a host.
  What we sometimes need is to CI a whole ecosystem. Can we templatize what we need to?
  VPCs can give us some of that - except for the expensive stuff.
* () -> instrumenting monitoring is a kind of testing
* () -> performance monitoring is a kind of testing
* () -> service availability monitoring is a kind of testing
* () -> vulnerability scanning is a kind of testing

How Can We Do Test Driven Development?
--------------------------------------

When we are doing something new involving code:

* Do TDD with unit tests as we write. 
* Get it running.

when we are doing something new (to us) involving configuration

* figure it out
* Get it running

Now is where our DevOps-ish testing comes in:

* Write the behaviorial, instrumenting, performance, service, vulnerability 
  tests
* Automate the deploy,  using 'side-effect' TDD. Not as valuable as "real" TDD
* run integration tests. 
* Now run your tests. Note this doesn't fit the TDD cylce.

When we "short-cut" the TDD cycle this way,
have to remember what we have done.
We have to remember to go back and run the full suite on a fresh rebuild.
We can automate this or enforce it, but it must be done.

What kind of testing frameworks can we use?
-------------------------------------------

We can keep using `pytest`, or `x-unit`, or whatever your comfortable with.

And there is nothing wrong with `serverspec`, `goss`. 
Our problem, once we start testing cross-host behavior
(can I reach host B from host A on port 1433?), 
is how to organize our tests?

We are testing *against* host B, but we need to run the test *on* host A. 

The tests *about* host B should be in a `host_B` directory,
but they may be pushed out and run on a variety of different hosts.
If they are meant to be living tests,
they may need to be templatized to be pushed to hosts in different environments.

We need to rethink testing in prod
----------------------------------------

* Mentioned the templatizing problem. 
* scaling, load test
* You can't change one thing in complex systems

It's not that we are going to *not* test before we go to prod.
It is that we are going to *continue* testing once we reach prod.

And we finally have the kinds of tools we need to test in prod.
We have feature flags. 
We have easy-ish instrument, performance, service availability monitoring.
And pretty decent vulnerability scanning.
