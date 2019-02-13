Preventing Cross Site Scripting Vulnerabilities Overview
========================================================

.. contents::
   :depth: 2
   :local:

Overview
--------

The following Preventing Cross Site Scripting (XSS) documentation is mainly dedicated to the issue of Cross Site Scripting XSS, which is just one of the `oWASP Top 10 Security
Vulnerabilities <https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project>`__.

Top Resources
-------------

-  `Preventing Cross Site Scripting (XSS) Vulnerabilities <preventing_xss.html>`__ (**Start with this!**): Contains general information and edx-platform specific help.

-  `Preventing XSS in Django Templates <preventing_xss_in_django_templates.rst>`__: If and when you are using Django Templates

-  `Preventing XSS in React <preventing_xss_in_react.rst>`__

-  Good article on `Input Validation in Python <https://ipsec.pl/python/2017/input-validation-free-form-unicode-text-python.html>`__. You must still use proper escaping on output!

Stripping Harmful HTML Tags
---------------------------

There are times when you don't want to see escaped HTML, but instead want to either:

-  Strip all HTML tags from your data, or

-  Strip all HTML tags except safe HTML tags (e.g. "<br />") from your data.

In both cases, we use a library called bleach.

Mako filters for bleaching
~~~~~~~~~~~~~~~~~~~~~~~~~~

At the time of writing this, we do not yet have Mako filters for bleaching.  However, that would be very useful and was `detailed and requested on this PR <https://github.com/edx/web-certificates/pull/55#discussion_r156088103>`__. If you need this, please do the following:

1. See if this was already added, 

2. If not, implement it and add to the `xss-lint repo <https://github.com/edx/xss-utils>`__.

3. Update this documentation to detail using the filters.

Strip all HTML tags
~~~~~~~~~~~~~~~~~~~

You would typically do this when people have entered HTML tags inside a field in the past, and you no longer want to support HTML, but you also don't want escaped HTML tags to start appearing on the page.

Here is an \ `example using bleach to strip all
tags <https://github.com/edx/edx-platform/blob/a864b450a889df77f1c7379271dc9a80b3c1a8ee/lms/templates/courseware/progress_graph.js#L76>`__.

Strip all but safe HTML tags
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You would do this if you in fact want to allow a user to be able to use certain simple HTML tags, like "<br />", in their input.  Use this sparingly.  It is much simpler to deal with plain text fields.

Here is an \ `example using bleach to only allow basic/safe supported
tags <https://github.com/edx/edx-platform/blob/e8a36957b1f732974260e7b9b42b9c25148b492c/common/lib/capa/capa/inputtypes.py#L792>`__.

In addition to adding in the HTML to your data, you will probably need to turn off HTML escaping when outputting this data inside a template. The following is an example of this in Mako.

.. code::

    # Sample Mako template with page level HTML-escaping on by default

    # Expression before:

    ${title}

    # Expression after:

    # Title was cleaned with bleach and is safe.
    ${title | n, decode.utf8}

XSS Testing
-----------

XSS Linting is performed as part of the jenkins/quality tests on Jenkins.

If you click the "Details" link next to the jenkins/quality check in GitHub, it will bring you to Jenkins.  Inside the quality build in Jenkins, you can get the following information:

-  Summary Details

   -  The "Quality Report" link on the left hand navigation contains the following:

      -  The "xsscommitlint" tab provides the number of violations in any file you added/modified in your commit, including previously existing violations.

      -  The "xsslint" tab provides the number of violations (per violation type) across the entire platform.

-  Failure Details

   -  The "Console Output" link on the left hand navigation will provide details of the failure.

      -  In the event of an XSS linter failure, the console output will state specifically which violation type went over the threshold and by how much.

   -  XSS failures mean that your commit bumped the number of violations over the current threshold.  This is most likely due to introducing a line of a code with a violation.  It is sometimes, but rarely, due to a revert of someone else's commit which had previously reduced the number of violations.

-  Detailed Reports

   -  XSS Commit Lint Report

      -  This report provides details of of violations in any file you added/modified in your commit, including previously existing violations.

      -  To see this report, you can append the following url to the end of your Jenkins job url:

..

   artifact/edx-platform/reports/xsscommitlint/xsscommitlint.report/*view*/

-  If you have a failure due to a threshold, you can use this report to drop below the threshold.

-  XSS Lint Report

   -  This is a platform level report, and is not generally useful.

To run these tests outside of Jenkins, see this \ `XSS quality tests <http://edx.readthedocs.io/projects/edx-developer-guide/en/latest/testing/code-quality.html?highlight=run_quality#safe-code>`__ documentation in Read the Docs.

XSS Linter
----------

:ref:`XSS Linter`

-  Documents xss linting as part of the quality build in edx-platform.

-  Includes detailed help for each type of linting violation.

To test files or templates outside of edx-platform, as a temporary hack, you can temporarily copy the file(s) under edx-platform and use the XSS Linter documentation above to run it against individual files or directories.

Code Reviews and XSS Linter Violations 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In certain cases, the quality check will fail in Jenkins when a violation is introduced that brings the number of violations of a particular type over its current threshold.

Jenkins will not fail if the XSS Commit Lint Report (see above) contains violations.  During a code review, if there are violations, here are the steps to be taken:

1. Bring awareness to the violations and to this process.

2. Reviewer and reviewee should discuss and choose one of the following approaches:

..

   Try to always use separate commits for safe template work to keep your options open during the PR process. You may consider keeping them separate even when squashing in case they introduce a separate issue.

   1. Fix all violations as part of this PR.

   2. Fix violations that are simple for fix and/or simple to test along with the current PR.

   3. Create a PR for follow-up work where appropriate.

   4. Determine that the violations are too far removed and leave this important work to some other developer.

These violations are technical debt that we all share as maintainers of the platform, and any help to reduce this debt is greatly appreciated.
