# circle-tag-filters
CircleCI is being weird. Time for a test.


Latest release: v0.3.0

---

This shows an issue with Sourcecred's current tag deployment.

https://github.com/sourcecred/sourcecred/blob/5727a831a92418d9247d2bd7dffd0707e78f6d75/.circleci/config.yml#L94-L114

```yml
      - docker/publish:
          # ...
          requires:
            - test
          filters:
            branches:
              ignore: /.*/
            tags:
              only: /^v.*/
          # ...
```

This job is intended to run only on tag releases with a `v` prefix.
However it has a requirement on a previous `test` job.

Looking at: https://circleci.com/docs/2.0/workflows/#executing-workflows-for-a-git-tag

> ### Executing Workflows for a Git Tag
>
> CircleCI does not run workflows for tags unless you explicitly specify tag filters. Additionally, if a job requires any other jobs (directly or indirectly), you must use regular expressions to specify tag filters for those jobs. Both lightweight and annotated tags are supported.

In other words, our deployment job would never run. As we require `test` to run first.<br>
And `test` has no explicit filter to run on tag pushes. So it implicitly **won't**.

![facepalm](https://media.giphy.com/media/SEp6Zq6ZkzUNW/giphy.gif)

To fix, we need to copy the test job and set it's filter so it will run on tag pushes.
At which point we might as well call this a different workflow.

```diff
workflows:
  version: 2.0
  commit:
    jobs:
      - test
      - deploy_dryrun:
          requires:
            - test
          filters:
            branches:
              ignore:
                - master
      - deploy_dev:
          requires:
            - test
          filters:
            branches:
              only: master
+
+  tagged-release:
+    jobs:
+      - test:
+          filters:
+            branches:
+              ignore: /.*/
+            tags:
+              only: /^v.*/
      - deploy_tag:
          requires:
            - test
          filters:
            branches:
              ignore: /.*/
            tags:
              only: /^v.*/
```
