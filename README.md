# agent-nav-dbx
Burcu Huff Databricks Component  - Capstone Project '25 Agent Nav Simulator

1. What “Databricks-managed serverless” really means

When I wrote:

“Because this workspace is Databricks-managed serverless, connecting directly to your personal AWS S3 bucket with IAM roles is a bit more work…”

I was talking about this specific situation:

You created a Databricks-managed trial workspace (the “Try Databricks / quickstart” option).

Compute is serverless + Databricks-managed AWS account, not your AWS account.

Your S3 bucket (agent-nav-logs-dev etc.) lives in your personal AWS account.

So from Databricks’ point of view, your S3 bucket is in a different AWS account. That’s why we started talking about:

Cross-account IAM role

External location / storage credential setup

Which is 100% doable, but:

It’s overkill for a capstone,

And easy to derail you with IAM yak-shaving.

That’s why I suggested the “export JSON → upload to Databricks” shortcut.

2. Other ways besides serverless (conceptually)

There are basically 3 big modes of “how Databricks runs compute”:

Serverless (Databricks-managed)

Databricks owns the AWS account & networking for compute.

You get “just works” clusters/SQL warehouses.

Accessing your AWS resources → cross-account setup.

Classic / All-purpose clusters in your AWS account

You choose “Use your existing cloud account” during workspace setup.

Databricks deploys VPC, EC2, IAM inside your AWS account.

Then:

You can give clusters an instance profile linked to an IAM role with s3:* on your bucket.

Or you define external locations that trust roles in the same account (simpler than cross-account).

This is the “traditional Databricks on AWS” setup companies use.

SQL-only / Unity Catalog-heavy setups

Heavier on external locations / storage credentials.

You define S3 locations once, give them a storage credential (IAM role).

Then you create tables on top of them.

Still boils down to: where is the workspace’s IAM role and how does it trust your bucket?

So yes, if you weren’t using the trial serverless workspace, there would be a much more “normal” pattern:

Workspace in your AWS account → give clusters an IAM role → that role has access to s3://agent-nav-logs-dev/* → directly spark.read.json("s3://...").

3. Realistic options for you right now

Given your situation (personal Databricks trial + personal AWS):

You basically have 3 realistic paths:

Option A — Stick with serverless + export/upload (simplest, lowest risk)

What we already discussed:

In AWS:

Download a small sample of runs/... JSON files from agent-nav-logs-dev.

In Databricks:

Upload them via “Add Data” → “Upload File” or use DBFS upload.

Read them with:

df = spark.read.json("dbfs:/FileStore/agent_nav_sample/")


Build:

A nice EDA notebook,

Some metrics by website / category / stealth mode,

Maybe 1–2 predictive features or a simple model.

✅ Pros:

No IAM wrestling.

Perfectly valid for your capstone: you’re showing analytics, experiment design, and dashboards, not infra wizardry.

Option B — Create a new non-serverless workspace tied to your AWS account

This is the “Use your existing cloud account” path you saw in the screenshot.

High-level steps (don’t worry, we don’t have to do this now, just explaining):

In Databricks:

Choose “Use your existing cloud account”.

Databricks gives you a CloudFormation template / permissions to set up.

In AWS:

Run the provided CloudFormation stack.

This creates:

A VPC,

IAM roles,

Databricks workspace infra in your account.

In Databricks (after workspace is ready):

Create a cluster with an instance profile that uses an IAM role that can read from agent-nav-logs-dev.

Then from a notebook:

df = spark.read.json("s3a://agent-nav-logs-dev/runs/")


✅ Pros:

“Real world” Databricks-on-AWS pattern.

Great for learning full-stack data engineering.

⚠️ Cons for you right now:

More setup time.

Easy to get lost in IAM/CloudFormation when your goal is to ship the capstone and tell a clean story.

Option C — Serverless + proper cross-account role (advanced but possible)

Even with serverless, we could go full “production style”:

In your AWS account:

Create an IAM role that:

Trusts the Databricks serverless AWS account (they document the account ID + external ID pattern).

Has S3 GetObject permissions on your bucket.

In Databricks:

Create a storage credential that uses that IAM role.

Create an external location pointing to s3://agent-nav-logs-dev.

Grant privileges.

Then:

CREATE TABLE agent_runs
USING json
LOCATION 's3://agent-nav-logs-dev/runs/';


✅ Pros:

Very “enterprisey”.

Great to mention in future if you actually set it up.

⚠️ Cons:

More moving parts (trust policy, external IDs, external locations, Unity Catalog configuration).

Much more than you need to prove Databricks analytics for the capstone.

4. So what should you do?

Given:

You already have a Databricks serverless workspace.

We’re under capstone time pressure.

Your goal is to show thoughtful analytics and experiment design, not to flex IAM.

I strongly recommend:

✅ Stay with serverless and do the “export sample JSON → upload to Databricks” flow.

You can still tell a great story:

“In production, logs live in S3.”

“For analytics, we sampled logs into Databricks for exploration and modeling.”

“In a future iteration, we’d wire up a direct external location with cross-account IAM.”

You’ll still get to:

Design metrics: success rate, token cost, time-to-success.

Slice by:

website category (.gov vs .com vs .edu),

model,

stealth vs non-stealth,

device/profile.

Build visuals + maybe a simple model.

That’s 100% Data Science / ML Systems and completely valid for a MIDS capstone.
