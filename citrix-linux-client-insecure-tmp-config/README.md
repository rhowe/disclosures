# Citrix Linux client insecure temporary ICA file creation (CVE-2023-24486)

The Citrix Linux client writes session configuration to a world-readable file
in /tmp. This file contains session connection details, including credentials.

Vendor advisory: [CTX477618](https://support.citrix.com/article/CTX477618).

# Software affected

- Citrix Workspace App for Linux versions 2212.

Other versions are likely affected.

I have not tested clients for other operating systems.

The issue is partially resolved in Citrix Workspace App for Linux 2302.

# Context

When connecting to a Citrix session via a web browser such as Firefox on Linux,
typically you access a web application known as Citrix Storefront. This
provides clickable icons for the applications and remote desktop sessions
available to you.

When you click on one of these, your browser is instructed to open a URL of the
form `receiver://.....` which is handled using `/opt/Citrix/ICAClient/util/ctxwebhelper`.
`ctxwebhelper` parses the URL and uses the decoded information to make a HTTP
GET request to the remote server for an 'ica' file, which contains the
connection details necessary to launch the Citrix client software,
`/opt/Citrix/ICAClient/wfica`.

The ICA file contains details such as the server hostname and temporary session
credentials needed to authenticate the session.

# The issue

`ctxwebhelper` writes the retrieved ICA file to `/tmp/launch.ica`. This file is
written with insecure file permissions allowing it to be read by any user of
the client device.

Any user of the client device can therefore obtain the session credentials by
waiting for this file to be created and reading it. Once this is done, it is
possible to initiate a session as the targeted user.

This can be demonstrated by running the following shell snippet and then
connecting to a Citrix session:

    while ! cat /tmp/launch.ica 2>/dev/null; do : ; done

When the ICA file is written to disk, the above shell snippet will read it and
dump it to standard output.

In addition, should the Citrix client crash the file may be left behind after
the process exits.

# Vendor response

Citrix acknowledged the bug, confirmed it as a security vulnerability and issued
an updated client in February 2023. A CVE was assigned.

Citrix security bulletin
[CTX477618](https://support.citrix.com/article/CTX477618) contains their
writeup.

# Workaround

There is no effective workaround for this issue, as the ICA file path is
hardcoded in ctxwebhelper.

If there are alternative ways to obtain the ICA file which do not depend on
receiver:// URLs it is possible they are unaffected by this issue.

Upgrade to version 2302 or later.

# Incomplete fix

The updated made in the February 2023 Citrix client is only a partial fix - the
updated client first tries to write the retrieved ICA file to the `.ICAClient`
directory in the user's home directory but if that fails for any reason, the
client falls back to the previous insecure behaviour of writing it to /tmp,
world-readable.

This can be demonstrated by creating a non-writable `.ICAClient` directory and
then launching a connection:

    mkdir -p ~/.ICAClient
    chmod u-wx ~/.ICAClient

I informed Citrix of this on 29 Oct 2023 but no update was forthcoming.

# Timeline

2022-10-12: Issue disclosed to Citrix via email to secure@citrix.com

2022-10-12: Citrix acknowledges receipt of the report, assigns identifier
`CASE-8312`.

2022-10-14: Citrix request additional information to help them reproduce the
issue. I responded the same day.

2022-10-20: Citrix confirm they can reproduce the issue and that they consider
it to be a security vulnerability. "remediation [..] may take a number of weeks
to address".

2022-12-11: Requested an update from Citrix.

2022-12-12: Citrix said they'd get back to me.

2022-12-15: Citrix confirm the fix has been completed but will not make the
cutoff for January release. A release containing the fix is scheduled for
February 2023. Citrix request disclosure be held until after then. I replied to
say OK.

2023-02-01: Citrix Workspace App for Linux version 2302 is released containing
a fix for this issue.

2023-02-13: Citrix request permission to include my name as credit in their
security bulletin.

2023-02-14: Citrix publish
[CTX477618](https://support.citrix.com/article/CTX477618), with summary
information regarding the vulnerability and directing users to the new version
of the client.

2023-02-14: I reply to say I'm OK with being credited by name.

2023-03-10: Writeup published.

2023-10-29: Contact Citrix to say the issue was not fully resolved, with
publication set at 45 days, 13 Dec 2023

2023-12-15: Published update detailing the incompleteness of the fix.
# Author

Russell Howe. [Github](https://github.com/rhowe) [Twitter](https://twitter.com/rhowe212).
