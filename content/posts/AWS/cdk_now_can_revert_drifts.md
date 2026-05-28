---
title: "CDK now can revert drifts"
date: 2026-05-27T00:00:00+02:00
lastmod: 2026-05-28T22:55:51+02:00
draft: false
---

AWS CloudFormation's behavior with drifts was always a bit weird, but in November last year they announced [drift-aware ChangeSets](https://aws.amazon.com/blogs/devops/safely-handle-configuration-drift-with-cloudformation-drift-aware-change-sets), which felt like the missing puzzle piece all along. These allow you to revert any changes that were made outside of CloudFormation — for example, if you changed a property in the UI. Previously, you were able to detect them, but you had to do the tedious work yourself to bring the resources back to the state your code represents.

Now they added a flag  `--deployment-mode REVERT_DRIFT`, which you can pass to your ChangeSet creation command. Once executed, any detected drift gets automatically reverted. This is basically the operating mode Terraform always had, and I envied Terraform for it all the time. This mode ensures that your infrastructure code always stays the single source of truth, which makes further development and deployments a lot more predictable. Now you have the choice in CloudFormation to use either the classic or the new mode, whichever you prefer. So I was hyped that this feature would come to CDK, my favorite IaC framework, eventually as well.


## The path to drift-aware ChangeSets in CDK {#the-path-to-drift-aware-changesets-in-cdk}

Around half a month after the CloudFormation announcement, on December 9th 2025, [a feature request](https://github.com/aws/aws-cdk/issues/36347) was raised on the GitHub repository of CDK. Two days later there was an answer already, but nothing more. Over the course of the next month the feature request gained 24 upvotes and got promoted to a p1 priority. It shouldn't be long until this gets implemented. Well, afterwards nothing happened. I also have to confess that I lost track and didn't catch that there was [a pull request](https://github.com/aws/aws-cdk-cli/pull/1127) opened by aki-kii on February 10th, 2026. It took three more weeks until March 6th to get approved and be merged into main. On the 9th of March the release containing the new `--revert-drift` flag for the CDK CLI was released with version 2.1110.0 ... and I completely missed it. Only today, when I was randomly looking for updates on this, was I surprised to find it had already shipped.

Maybe it got short shrift compared to the other big new features like mixins that got a lot more coverage from AWS. At least it seemed like it, because when I googled around today, the only thing I found was this PR and a post on dev.to mentioning this alongside said other features. I went ahead, tested it, and it was exactly what I had wished for.


## How to work with drifts in CDK now {#how-to-work-with-drifts-in-cdk-now}

So how does it work exactly? Let's try it out in a short example. I've set up a default CDK app where the stack file looks like this:

```typescript
import * as cdk from 'aws-cdk-lib/core';
import { Construct } from 'constructs';
import * as sqs from 'aws-cdk-lib/aws-sqs';

export class CdkDriftExampleStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const queue = new sqs.Queue(this, 'CdkDriftExampleQueue', {
      visibilityTimeout: cdk.Duration.seconds(300)
    });
  }
}
```

Once deployed via `cdk deploy`, we see our queue with a visibility timeout of 300 seconds (5 minutes)

{{< figure src="/ox-hugo/cdk_now_can_revert_drifts_sqs_queue.png" >}}

Let's change the visibility timeout to 30 minutes in the UI

{{< figure src="/ox-hugo/cdk_now_can_revert_drifts_sqs_queue_change.png" >}}

Once we save the change in the UI, let's check what our diff looks like:

```bash
$ cdk diff
start: Building CdkDriftExampleStack Template
success: Built CdkDriftExampleStack Template
start: Publishing CdkDriftExampleStack Template (current_account-current_region-216f9038)
success: Published CdkDriftExampleStack Template (current_account-current_region-216f9038)
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --method=template to use a less accurate but faster template-only diff)

Stack CdkDriftExampleStack
There were no differences


✨  Number of stacks with differences: 0
```

As you can see, there are no differences to deploy. This is CloudFormation's default behavior, which in comparison to Terraform, ignores changes from elsewhere. Doing a `cdk deploy` afterwards would also yield no change.

Now let's check again with CDK's drift command:

```bash
$ cdk drift
✨  Synthesis time: 2.09s
Stack CdkDriftExampleStack
Modified Resources
[~] AWS::SQS::Queue CdkDriftExampleQueue CdkDriftExampleQueueEDCD7478
 └─ [~] /VisibilityTimeout
     ├─ [-] 300
     └─ [+] 1800

1 resource has drifted from their expected configuration

✨  Number of resources with drift: 1
```

As you can see, drift detection reveals these manual changes. Now let's use the new `--revert-drift` flag during a cdk deploy:

```bash
cdk deploy --revert-drift

✨  Synthesis time: 1.75s

CdkDriftExampleStack: creating CloudFormation changeset...
Changeset arn:aws:cloudformation:eu-central-1:123456789:changeSet/cdk-deploy-change-set/76acc120-21b4-4298-96f8-1911f4a07ff1 created and waiting in review for manual execution (--no-execute)
CdkDriftExampleStack: deploying... [1/1]

 ✅  CdkDriftExampleStack

✨  Deployment time: 74.39s

Stack ARN:
arn:aws:cloudformation:eu-central-1:123456789:stack/CdkDriftExampleStack/3fa073f0-59ec-11f1-b387-0276a3360ee7

✨  Total time: 76.14s
```

This time `cdk deploy` actually made a change. Looking back into the UI or calling `cdk drift` again will show that the visibility timeout is back to 300 seconds / 5 minutes.


## Some more shenanigans with reverting drifts {#some-more-shenanigans-with-reverting-drifts}

Before, we had an easy case where we only had a drift, but how does `--revert-drift` handle more complex situations? Let's go through two more examples.

First, we change the field by hand again to 30 minutes, but this time we also update the same attribute in code to 1 minute (60 seconds).

{{< figure src="/ox-hugo/cdk_now_can_revert_drifts_double_change_same_attribute.png" >}}

We get the following from `cdk diff`

```bash
$ cdk diff
[~] AWS::SQS::Queue CdkDriftExampleQueue CdkDriftExampleQueueEDCD7478
 └─ [~] VisibilityTimeout
     ├─ [-] 300
     └─ [+] 60
```

and `cdk drift` respectively

```bash
$ cdk drift
[~] AWS::SQS::Queue CdkDriftExampleQueue CdkDriftExampleQueueEDCD7478
 └─ [~] /VisibilityTimeout
     ├─ [-] 300
     └─ [+] 1800
```

Deploying now with `cdk deploy --revert-drift` will set the _visibilityTimeout_ to 60, which makes sense, because CDK reverts the change back to what we have written in code.

For the second example, we are back at our initial value of 300 seconds for the _visibilityTimeout_, but now we also change a second attribute, the _retentionPeriod_

{{< figure src="/ox-hugo/cdk_now_can_revert_drifts_double_change_different_attribute.png" >}}

Again, diff and drift show two different changes

```bash
$ cdk diff
[~] AWS::SQS::Queue CdkDriftExampleQueue CdkDriftExampleQueueEDCD7478
 └─ [+] MessageRetentionPeriod
     └─ 100
```

```bash
$ cdk drift
[~] AWS::SQS::Queue CdkDriftExampleQueue CdkDriftExampleQueueEDCD7478
 └─ [~] /VisibilityTimeout
     ├─ [-] 300
     └─ [+] 1800
```

Looking at the resource in AWS now, we see both attributes got changed at the same time.

{{< figure src="/ox-hugo/cdk_now_can_revert_drifts_double_change_different_attribute_result.png" >}}

The drift and the diff change were applied within the same operation. This means that, while you currently need two commands to see all the changes to your resources — and maybe there will be an additional argument for `cdk diff` at some point — `cdk deploy --revert-drift` handles both types of changes in a single operation. If you intend to use this feature in your deployment pipelines, it's enough to extend your existing `cdk deploy` command with this new flag.


## Where to go from here {#where-to-go-from-here}

What I really like about how CloudFormation and CDK went about this change is that they made it an optional extension to the existing CloudFormation API. Nothing changes if you are happy with the previous behavior. But if you want a more Terraform-like behavior, you can have it now.

In the coming days I can see myself extending CI checks with an additional `cdk drift` step next to the usual `cdk diff`, which writes the drift result into a pull request comment next to the diff output. `cdk deploy` in your actual deployment pipeline will be replaced by `cdk deploy --revert-drift`. With this you get visibility into whether something changed outside your code. You also get much more confidence that the code you are looking at is actually what is deployed in your AWS environment.

I was surprised that this feature didn't get more coverage so far. That is also where my motivation for writing about this, is coming from. I at least was waiting a long time for this. Actually, I was even asking the presenters of ["AWS infrastructure as code: A year in review"](https://youtu.be/fROlLTMRi0Y?si=Upet5HGKbD0OYMZ_) at the re:invent 2023 whether they planned to change their deployment behavior to something more similar to Terraform. They already hinted that they were planning something in this direction, but it would be an optional setting and you would be able to choose to deploy in the mode you'd prefer. Two years later, we got exactly what they hinted at back then and I'm very happy with this extension.
