---
layout: post
title:  "AWS - Cloud Security [chapter 0x1] - Identifying AWS Account ID from a public S3 - PWNEDLABS (Different Approach)"
date:   2024-01-05 13:44:25 +0700
categories: "AWS_Cloud_Security"
---

Hello world!

<h2>Background</h2>

Before we getting start with the content, I would like to mentioned that this post is originally come from the following ([lab][lab]) created by pwnedlabs. This post is inspired from the following youtube comment. 

So little bit of background of the lab, the lab itself taught you how to leverage a misconfigurate public s3 bucket that could lead to exposing AWS Account ID using tools such as s3-account-search. The lab gives you 2 options to do the task either using the access account and secret key that provide by the pwnedlabs or you can setup access account and secret key by yourself through your AWS account.

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/youtube_comment.png){:height="200px" width="800px"}
{:refdef}

The first task is already cover by Afshan - AFS Hackers Academy you can also check youtube video from this ([link][link]) and the original youtube comment also posted in this video. However, as I looking around the internet and some write up, I don't find any blog that talk about the second option or is it me that not looking for it hard enough. Thus, in this post I will share it to you how to setup the IAM user that can be used to running s3-account-search.

Finally, let me also emphasize it again that knowing AWS Account ID is not consider a vulnerability nor a security flaw, however, as stated in the lab that knowing the account ID of an organization you are assessing the security of could be helpful for further analysis. 

This post will be 

<h2> Part 1 - IAM Setup </h2>

The first part is pretty straightforward as you need to create a IAM user that have sts:AssumeRole and S3 bucket policy which are s3:GetObject and s3:ListBucket. In this case, I created one new user named "leaky_user_bucket", in this case I do not attached any permission or role yet, this will come later. 

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/create_user_iam.png){:height="300px" width="900px"}
{:refdef}

After that I create access key for this user 

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/create_access_key.png){:height="300px" width="900px"}
{:refdef}

Next is to create two policies that I have mentioned earlier and attached this into the "leaky_user_bucket"

When creating the policy I just straight to specify permissions via JSON mode in policy editor

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/json_mode.png){:height="100px" width="1000px"}
{:refdef}

Put the following detail, I changed the aws account id based on the user "leaky_user_bucket" and for the role you can put any name but in this case I will put "LeakyBucket"

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/sts_assume_role.png){:height="300px" width="1500px"}
{:refdef}

After that you saved and named the policy anything you want while in this case I put the policy name as "assume_role_policy_pwned". Finally you create the S3 bucket policy like this:

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/s3_bucket.png){:height="400px" width="1200px"}
{:refdef}

Again for the naming you can choose anything you want while in this case I put the name as "mega_big_bucket_leak". At the end of this part you will have two policies like this:

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/list_policies.png){:height="200px" width="1200px"}
{:refdef}

<h2> Part 2 - Rabbithole </h2>

Ok! I think this part will be the one that make most people confused because based on the lab after assigning these two policies to the user, like this:

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/add_policy.png){:height="400px" width="1800px"}
{:refdef}

It will be sufficient but its not! if you try to run the s3-account-search program it will get the following error:

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/error.png){:height="50px" width="1800px"}
{:refdef}

To resolve this issue we need to add one more key component which is "role" for S3 bucket

<h2> Part 3 - Resolve </h2>

Go to "role" section to create a new role for the user "leaky_user_bucket" for the policy inside the role we will used "custom trust policy" like this: 

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/custom_role.png){:height="200px" width="1800px"}
{:refdef}

Put the following policy

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/leaky_bucket_role.png){:height="400px" width="1800px"}
{:refdef}

ok this is very important the name of the role need to be the same as the one you created in sts:AssumeRole if you don't it will give you the same error. Finally, you put the ARN of your aws account that you have created earlier in this case it will be the "leaky_user_bucket" by doing this we assing this role to our current user.

Then you can run the s3-account-search tool again and there you go it will give the account id:

{:refdef: style="text-align: center;"}
![SETUP_PREVIEW]({{site.url}}/blog/img/identify_s3_id/result.png){:height="400px" width="1500px"}
{:refdef}

[lab]: https://pwnedlabs.io/labs/identify-the-aws-account-id-from-a-public-s3-bucket
[link]: https://www.youtube.com/watch?v=jXleMOWvq80
