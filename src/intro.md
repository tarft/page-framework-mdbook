# Page framework docs (2026-08-23)

Status:

Most functionality of the `DescribePropositions` task done (formally known as just `Propositions` -- renamed to establish convention `DescribeXyz` for all NLP tasks). Currently just fixing some bugs that prevent the task from fully running.

Previously I implemented a very basic dummy task at `AddNumbers`.

Most of the cosmetics not pulled over from the old framework yet, so it's rough on the eyes.

Currently just displays a single page. "Non-core features" like loading instances / LTI / history / logging not touched yet, probably a lot of small TODOs I'm forgetting. Also probably needs a safety audit / stress test at some point.

## Installation

Clone the following repos: https://ls1-gitlab.cs.tu-dortmund.de/ILTIS/pageframework

Checkout the `page-framework` branch on utils.

For development, I recommend setting up an entirely separate IntelliJ project from the "old" framework, with separate tmp directories. Set up Jetty / GWT configurations similar to the old ones, just with `Runner` replacing all previous instances of `LogicWeb`.
