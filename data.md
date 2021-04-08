Pixel test reference image storage
----------------------------------

- Pixel tests need reference images.  Those images are maintained by
  developers, as part of the tests.  Candidates for new reference
  images are produced by machine (by running the tests), but
  developers decide which ones to make part of the tests.  Developers
  might also edit the reference images directly to mark regions that
  should be ignored.

- Changes to reference images need to be reviewed and approved, like
  other changes to code.  For example, when updating Patternfly or
  other node modules, the bots currently ask the developers to review
  certain Cockpit pages manually.  This would be replaced/augmented by
  reviewing changes to the reference images of the pixel tests.

- Github can diff PNGs right in the pull request, but one can not
  attach comments to them.

- Ideal: just commit the reference images into the
  cockpit-project/cockpit repository.  The issue with this is data
  size.  While each individual PNG is smallish (5k ish), they might
  churn a lot.  I.e., size of a checkout will not be an issue, but
  infinite history of reference images will be.

- Issues with data size: naive repository cloning will be slow,
  github.com might not be happy (above 1 GiB is not recommended),
  local repositories will be big.

- Purging old reference images from cockpit-project/cockpit means
  rewriting history and that is not really an option.  Github will
  probably not be able to associate commits with pull requests
  anymore, for example.

- Git-lfs can arrange things so that files are not actually in the git
  repo (just a pointer of a few hundred bytes), and does so very
  transparently.  Github.com has full support for git-lfs: it has a
  LFS server and the UI transparently works with files that are stored
  in git-lfs.

- However, purging old reference images from the LFS server on
  github.com seems to be even harder than purging them from a plain
  git repo: One still needs to rewrite the commit history, and on top
  of that, to purge the actual LFS storage, one needs to delete the
  project and recreate it, which loses all issues etc.  I think.  It's
  not straightforward in any case.

- Without purging history, git lfs on github.com might become a money
  trap.  A free plan gets 1 GiB storage and 1 GiB per month download,
  This should be ok for about 200000 reference images, which might
  last us a couple of years.  The bandwidth cap is new and might
  strike at bad moments.

- We could use our own LFS server, and purge images from there.  This
  would be extra work, and we would very likely lose github.com
  integration.

- We need to run "git lfs pull" for a checkout that has been created
  before git-lfs was installed.  Such as with Cirrus and Packit.  Not
  a problem, I'd say.

- Thus, git-lfs would solve the client-side problems of infinite
  reference image history (slow clones), but not the server side
  (unlimited growth).

- Another option to solve the client-side issues with "partial git
  clones".  All reference images would go into
  cockpit-project/cockpit.  This will bloat the repository on the
  server, but local checkouts will be just as fast (or faster) as with
  git-lfs.  Github.com should allow us to bloat the server side repo
  way past 1GiB, so this option is also good for a couple of years.

- This is not transparent, however: It requires passing a (potentially
  super complex) --filter option to "git clone".  I have not found a
  way to have the server tell the client a default filter option.  If
  you omit this filter option, there will be pain.  We might not be
  able to easily modify how external CIs like Cirrus and Packit call
  "git clone".

- If we go with git-lfs on github.com and change our minds to
  something else, we can do that pretty easily.  The
  cockpit-project/cockpit repo at that time only has pointers and is
  small and nibmle.  Nobody needs to do anything special to work with
  that repository going forward.  Out LFS storage on github would be
  full at that point, probably, and I don't know if it can be emptied.

- If we go with images in cockpit-project/project and partial clones,
  we can never get away from partial clones without a history rewrite.
  The server side repo will stay impossibly bloated forever and people
  need to use partial clones forever to be able to survive it.

- Still another fork in the road: let's not commit reference images at
  all, but produce them on demand by running the tests in a special
  mode and retrieving the screenshots they produce.  (I missed this
  option all the time, thanks Garrett for making me see it.)  Lots of
  thinking needed here.

  => can't use github.com to directly review proposed changes to
     reference images
  => no (easy?) editing of alpha channel of reference images


Summary:

- It's possible to start with git-lfs on github.com until we have more
  experience with where want pixel testing to go.

- While we ramp up pixel testing (using git-lfs), we can work on
  implementing PR image reviews that don't need the files to be
  available to github.com

- Also, we can experiment with abstracting the storage location of
  reference images away from developers and once that is in place, we
  switch it to something that we are comfortable with.  We can give us
  a limit and say that we need to have this done before we have used
  200 MiB of our LFS storage, for example.
