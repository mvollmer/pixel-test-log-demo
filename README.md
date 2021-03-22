The Cockpit integration tests might soon be able to contain "pixel
tests".  Such a test will take a screenshot with the Browser and
compare it with a reference.  The idea is that we can catch visual
regressions much easier this way than if we would hunt for them in a
purely manual fashion.

## Adding a pixel test

To add a pixel test to a test program, call the new "assert_pixels"
function of the Browser class:

     ...
     self.browser.assert_pixels('#dialog button.apply', "button")
     ...

The first argument is a CSS selector that identifies the part of the
UI that you want to compare.  The screenshot will only include that
element.  The secon argument is a arbitrary but unique key for this
test point.  It is used to name the files that go with it.

For each such call, there needs to be a reference image in the
test/reference/ directory.

The easiest way to get a new reference image is to just run the test
once (locally or in the CI machinery).  It will fail, but produce a
reference image for the current state of the UI.

When you run the test locally, the new reference image will appear in
the current directory.  Just move it into test/reference/ and commit
it.  The next run of the test should be green.

When you run the tests in the CI machinery, you need to download the
new reference images when the test results directory.  There is a
script that can help with that.  (In
cockpit/test/common/update-reference-pixels, but it's very rough
still.)

If there are parts of the reference image that you want to ignore, you
can change the transparency (alpha channel) of those parts in the
image itself (with the GIMP, say).  Any pixel that is not fully opaque
will be ignored during comparison.

Here is a PR that adds two pixel test points to the starter-kit:

  [starter-kit#436](https://github.com/cockpit-project/starter-kit/pull/436)

## Debugging a failed pixel test

When making changes that change how the UI looks, some pixel tests
will fail.  The test results will contain the new pixels, and means
for directly seeing what has changed.

Here is a PR that makes the two pixel tests fail that had been added
in#436:

  [starter-kit#435](https://github.com/cockpit-project/starter-kit/pull/435)

As you can see, it has "pixels" links in the same place as the well
known "screenshot" links.  Clicking on it gets you to a page where you
can directly compare the previous and current UI.

As the author of the pull request, you can decide from there whether
these changes are intended or not.

For a intended change, see the next section.  For unintended changes,
you need to fix your code and try again, of course.

## Updating a pixel test

If you make a change that intentionally changes how Cockpit looks (and
most changes are like that, I'd say), you need to install new
reference images in test/reference/.

This is very similar to adding a new pixel test point.  Take the
TestFoo-testBasic-pixels.png that was written by the failed test run,
move it into test/reference, and commit it.  A local test run has
dropped it into the current directory, for a remote run it will be in
the test results directory.  (The update-reference-pixels script is
intended to help, but needs more love first...)

When a test writes a new Foo-pixels.png file for a failed test, it
will have the alpha channel of the old reference copied into it.  That
makes it easy to keep ignoring the same parts, but you can of course
change it if necessary.

## Reviewing a changed pixel test

Here is a second version of the starter-kit pull request from the
previous section:

  [starter-kit#438](https://github.com/cockpit-project/starter-kit/pull/438)

It has the same code changes, but now the reference images have been
updated as well, since the change in color was of course intended.

Now this PR needs to be reviewed, and the changed visuals need to be
approved.  Github can do image comparisons, so we will use that to
review changes to the reference images.  Click on the "Show rich diff"
button to see the image diff.

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

- The new update-reference-pixels script, pixeldiff.html, and log.html
  need some love.

- The pixel comparison machinery test results don't make sense for new
  references, probably.
