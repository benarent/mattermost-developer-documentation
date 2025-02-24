---
title: "Feature Flags"
heading: "Feature Flags and Mattermost Cloud"
description: "Feature flags allow us to be more confident in shipping features continuously to Mattermost Cloud. Find out why."
date: 2020-10-15T16:00:00-0700
weight: 3
subsection: Server
---


Feature flags allow us to be more confident in shipping features continuously to Mattermost Cloud. Feature flags also allow us to control which features are enabled on a cluster level.

# How to use

## Adding the feature flag in code

1. Add the new flag to the feature flag struct located in `model/feature_flags.go`.
2. Set a default value in the `SetDefaults` function in the same file.
3. Use the feature flag in code as you would use a regular configuration setting. In tests, manipulate the configuration value to test value changes, such as activation and deactivation of the feature flag.
4. Code may be merged regardless of setup in the management system. In this case it will always take the default value supplied in the `SetDefaults` function.
5. Create a removal ticket for the feature flag. All feature flags should be removed as soon as they have been verified by Cloud. The ticket should encompass removal of the supporting code and archiving in the management system.

### Feature flag code guidelines

- A ticket should be created when a feature flag is added to remove the feature flag as soon as it isn't required anymore.
- Tests should be written to verify the feature flag works as expected. Note that in cases where there may be a migration or new data, off to on and on to off should both be tested.
- Log messages by the feature should include the feature flag tag, with the feature flag name as a value, in order to ease debugging.

## Deploying to split.io

Deployments to the management system are overseen by the Cloud team. If you have any questions or need help with the process, please ask in the Cloud channel.

1. When ready to deploy the feature, create the feature flag (called a split) in split.io. The name of the feature flag should match the name of the split. Everything else can be left at defaults.
2. Once created, set the treatment values appropriately. The defaults of "on" an "off" can work for most feature flags.
3. When ready to deploy to cloud, set the targeting rules appropriately to slowly roll out as required. 

## Timelines for rollouts

The feature flag is initially “off” and will be rolled out slowly. Individual teams should decide how they want to roll out their features as they are responsible for them and know them best. Once we have split.io access for 2-3 people per team, the feature teams can enable/disable feature flags at will without needing to ask the Cloud team. 

**Note:** The steps below are an initial guideline and will be iterated on over time.

 - 1st week after feature is merged (T-30): 10% rollout; only to test servers, no rollout to customers.
 - 2nd week (T-22): 50% rollout; rollout to some customers (excluding big customers and newly signed-up customers); no major bugs in test servers.
 - 3rd week (T-15): 100% rollout; no major bugs from customers or test servers. 
 - End of 3rd week (T-8): Remove flag. Feature is production ready and not experimental.

For smaller, non-risky features, the above process can be more fast tracked as needed, such as starting with a 10% rollout to test servers, then 100%.
Features have to soak on Cloud for at least two weeks for testing. Focus is on severity and number of bugs found; if there are major bugs found at any stage, the feature flag can be turned off to roll back the feature.

When the feature is rolled out to customers, logs will show if there are crashes, and normally users will report feedback on the feature (e.g. bugs).

## Self-managed releases

For a feature-flagged feature to be included in a self-managed release, the feature flag should be removed.

Feature flags are generally off by default and self-managed releases do not contact the management system. Therefore feature flags that are not ready for a self-managed release will be automatically disabled for all self-managed releases.

## Testing

Tests should be written to verify all states of the feature flag. Tests should cover any migrations that may take place in both directions (i.e., from "off" to "on" and from "on" to "off"). Ideally E2E tests should be written before the feature is merged, or at least before the feature flag is removed.

# When to use

There are no hard rules on when a feature flag should be used. It is left up to the best judgement of the responsible engineers to determine if a feature flag is required. The following are guidelines designed to help guide the determination.

- Any "substantial" feature should have a flag
- Features that are probably substantial:
    - Features with new UI or changes to existing UI
    - Features with a risk of regression
- Features that are probably not substantial:
    - Small bug fixes
    - Refactoring
    - Changes that are not user facing and can be completely verified by unit and E2E testing.

## Examples of feature flags

< add some examples when we create them >

## FAQ

1. What is the expected default value for boolean feature flags? Is it `true` or `false`?
 - Definitely `false`. The idea is to use them to slowly roll out a feature. When the code is deployed, the feature flag is not enabled yet. See more details on feature flag rollout timelines [here](https://developers.mattermost.com/contribute/server/feature-flags/#timelines-for-rollouts).

2. Is it possible to use a plugin feature flag such as `PluginIncidentManagement` to "prepackage" a plugin only on Cloud by only setting a plugin version to that flag on Cloud? Can self-managed customers manually set that flag to install the said plugin?
 - Yes. If you leave the default "" then nothing will happen for self-managed installations. You can ask the Cloud team to set ``split.io/environment`` to a specific version.

3. How do feature flags work on webapp?
 - To add a feature flag that affects frontend, the following is needed: 
    1. PR to server code to add the new feature flag. 
    2. PR to redux to update the types. 
    3. PR to webapp to actually use the feature flag.

4. How do feature flags work on mobile?
 - To add a feature flag that affects mobile, the following is needed: 
    1. PR to server code to add the new feature flag. 
    2. PR to mobile to update the types and to actually use the feature flag.

5. How do we enable a feature flag for testing on community-daily and on Cloud test servers?
 - You can post in [~Developers: Cloud channel](https://community.mattermost.com/core/channels/cloud) with the feature flag name and what you want the Cloud team to set it to.

6. What is the environment variable to set a feature flag?
 - It is `MM_FEATUREFLAGS_<myflag>`.

7. Can plugins use feature flags to enable small features aside of the version forcing feature flag?
 - Yes. You can create feature flags as if they were added for the core product, and they'll get included in the plugin through the config.

8. Does it make sense to use feature flags for A/B testing?
 - This is something we're going to be evaluating using split.io. We've already implemented support for this in the server.

9. Do feature flag changes require the server to be restarted?
 - Feature flags don’t require a server restart unless the feature being flagged requires a restart itself.

10. For features that are requested by self-managed customers, why do we have to deploy to Cloud first, rather than having the customer who has the test case test it?
 - Cloud is the way to validate the stability of the feature before it goes to self-managed customers. In exceptional cases we can let the self-managed customer know that they can use environment variables to enable the feature flag (but specify that the feature is experimental).

11. How does the current process take into account bugs that may arise on self-managed specifically?
 - The process hasn’t changed much from the old release process: Features can still be tested on self-managed servers once they have been rolled out to Cloud. The primary goal is that bugs are first identified on Cloud servers.

12. How can self-managed installations set feature flags?
 - Self-managed installations can set environment variables to set feature flag values. However, users should recognize that the feature is still considered "experimental" and should not be enabled on production servers.
