---
layout: blog
title: Tip of the Week 69 - example42 Puppet Tutorial - Part 3
---

### example42 Puppet Tutorial - Part 3

This is the third post of a series of articles covering an introduction to Puppet.

In the [first post](https://example42.com/blog) I started with Puppet agent installation and how to use Puppet and Facter to analyze your system. Next topics have been the introduction to the Puppet programming language (DSL), how to setup the central Puppet master and how to connect Puppet agents to the Puppet master.

The [second posting](https://example42.com/blog) covered cover Puppet modules, code logic and variables and how to add external facts to your systems. Besides this I introduced parameters and the concept of separating code and data by using hiera.

This third part will explain how to make use of upstream Puppet libraries when describing your own infrastructure, how to best classify nodes and where to place the code.

At the last posting I will combine what I have shown and explain how to make use of the example42 [PSICK control repository](https://github.com/example42/psick.git), the [PSICK module](https://github.com/example42/puppet-psick.git) and the [PSICK hieradata](https://github.com/example42/psick-hieradata).

* Table of content
{:toc}

#### Technical Component (Library) Modules

#### Implementation Profiles

#### Business Use Case Role

#### Node Classification

#### The Puppet Control Repository

The upcoming posting will explain the concept of PSICK and how you can easily adopt it to your infrastructure.

Martin Alfke