The Cockpit integration tests might soon be able to contain "pixel
tests".  Such a test will take a screenshot with the browser and
compare it with a reference.  The idea is that we can catch visual
regressions much easier this way than if we would hunt for them in a
purely manual fashion.

## Preparing a repository for pixel tests

A pixel test will take a screenshot of part of the Cockpit UI and
compare it with a reference.  Thus, these reference images are
important and play the biggest role.

A large part of dealing with pixel tests will consequently consist of
maintaining the reference images.  At the same time, we don't want to
clog up our main source repository with them.  While the numer and
size of the reference images at any one point in time should not pose
a problem, we will over time accumulate a history of them that we are
afraid would dominate the source repository.

Thus, the reference images are not stored in the source reposirory.
Instead, we store them in a submodule-kind-of external repository.
That external repository doesn't keep any history can be aggressively
pruned.

The details of this storage repository are not nailed down yet.  It
might be a actual submodule with some sugar on top, or it might be
something eerily similar to a submodule.  In any case, the idea is
that developers are isolated from it via the new
`test/common/pixel-tests` tool.

That tool isn't nailed down yet, either.  But it's good enough to get
some real work done with it, and gain the experience necessary to
improve it.

In any case, a source repository needs to be prepared before it can
store reference images in a external storage repository.  The tool
will be able to do that, but right now it is still a manual process.
Please ask on IRC for help.

## Adding a pixel test

To add a pixel test to a test program, call the new "assert_pixels"
function of the Browser class:

     ...
     self.browser.assert_pixels('#dialog button.apply', "button")
     ...

The first argument is a CSS selector that identifies the part of the
UI that you want to compare.  The screenshot will only include that
element.  The second argument is an arbitrary but unique key for this
test point.  It is used to name the files that go with it.

For each such call, there needs to be a reference image in the
`test/reference/` directory.

As mentioned above, the `test/reference/` directory is very special
indeed, and needs to be carefully managed with the
`test/common/pixel-tests` tool.

First, make sure `test/reference` is up-to-date:

```
$ ./test/common/pixel-tests pull
```

Then you can add new reference images to it.

The easiest way to get a new reference image is to just run the test
once (locally or in the CI machinery).  It will fail, but produce a
reference image for the current state of the UI.

When you run the test locally, the new reference image will appear in
the current directory.  Just move it into test/reference/.  The next
run of the test should be green.

When you run the tests in the CI machinery, you need to download the
new reference images from the test results directory.  They show up as
regular screenshots.

If there are parts of the reference image that you want to ignore, you
can change the transparency (alpha channel) of those parts in the
image itself (with the GIMP, say).  Any pixel that is not fully opaque
will be ignored during comparison.

When you are done adding images to `test/reference`, push them into
the storage repository like so:

```
$ ./test/common/pixel-tests push
```

This `push` command will record a change to `test/reference` in the
main repository, and you need to commit this change.  This is how the
main repository specifies which reference images to use exactly for
each of its commits.

Here is a PR that adds two pixel test points to the starter-kit,
complete with reference images:

  [starter-kit#436](https://github.com/cockpit-project/starter-kit/pull/436)

## Debugging a failed pixel test

When making changes that change how the UI looks, some pixel tests
will fail.  The test results will contain the new pixels, and you can
compare them with the referene image right in the browser when looking
at the test logs.

Here is a PR that makes the two pixel tests fail that had been added
in #436:

  [starter-kit#435](https://github.com/cockpit-project/starter-kit/pull/435)

As you can see, it has "changed pixels" links in the same place as the
well known "screenshot" links.  Clicking on it gets you to a page
where you can directly compare the previous and current UI.

As the author of the pull request, you can decide from there whether
these changes are intended or not.

For a intended change, see the next section.  For unintended changes,
you need to fix your code and try again, of course.

## Updating a pixel test

If you make a change that intentionally changes how Cockpit looks, you
need to install new reference images in `test/reference/`.

This is very similar to adding a new pixel test point.  Take the
TestFoo-testBasic-pixels.png that was written by the failed test run,
move it into `test/reference`, push it to the storage repository with
`./test/common/pixel-tests push`, and commit `test/reference` in the
main repository.

A local test run has dropped the new reference image into the current
directory, for a remote run it will be in the test results directory
and the pixel comparison view has a link to it.

When a test writes a new TestFoo-testBasic-pixels.png file for a
failed test, it will have the alpha channel of the old reference
copied into it.  That makes it easy to keep ignoring the same parts.

## Reviewing a changed pixel test

Here is a second version of the starter-kit pull request from the
previous section:

  [starter-kit#438](https://github.com/cockpit-project/starter-kit/pull/438)

It has the same code changes, but now the reference images have been
updated as well, since the change in color was of course intended.

Now this PR needs to be reviewed, and the changed visuals need to be
approved.  But since the reference images are not stored in the main
repository, Github will not include them in the PR diff view.

Instead, the robots will automatically add a comment to a pull request
with a link to a page that allows reviewing the changed reference
images.

## Randomness

Some parts of the UI will change from one run to the next although the
code has not.  For example, if the UI displays the current time, or
the PID of some process, those values will potentially be different on
every run.  The idea is that we battle these things individually, as
they come up.  Maybe we ignore parts of the UI (by making the
reference image transparent in those areas), or maybe we can remove
the randomness in the test itself (by setting the time and date to a
specific value).

But there is at least one aspect where the browser itself adds
randomness: How a font glyph is rendered can change from one run to
the next, i.e., on one run all "S" characters look the same, and on
the next run, they still look all the same, but different from the
previous run.  The changes are small, of course, since it's still the
letter "S" from the same font at the same size.  Here the idea is to
tweak the image comparision algorithm to ignore this.  (Small changes
in intensity are allowed, but any change in color is rejected.)

## Open issues

- The pixel comparison view (pixeldiff.html) could be improved, I
  guess.

- Preparing a main repo is a manual task.

- test/common/pixel-tests isn't any good
