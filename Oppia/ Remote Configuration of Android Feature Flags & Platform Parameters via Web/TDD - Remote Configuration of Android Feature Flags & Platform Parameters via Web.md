# TDD: Remote Configuration of Android Feature Flags & Platform Parameters via Web

*Author(s): Nitish Kumar*  
*Date: 22 December 2025 (created)*  
*Quick links: [Section 1: WHAT](#bookmark=kix.tbma9pxzm6z0); [Section 2: HOW](#bookmark=id.19xs5lvnc44w)*  
*Product Requirements Doc (if applicable): N/A*  
*Other Relevant Docs (if applicable): [https://github.com/oppia/oppia/issues/21097](https://github.com/oppia/oppia/issues/21097)*   
*Google Doc Link:https://docs.google.com/document/d/1OyT2jfL-2K3gZ3weWLQhRtjxqfwkfcADM3I1xAObFas/edit?tab=t.0#heading=h.liiqbmwmz8j8*

# Approvals

| Reviewer | Role | Review Status  | Last updated |
| :---- | :---- | :---- | :---- |
| Brian Rodriguez | ~~Team lead for LEAP~~ Primary Reviewer | **Go (fully approved)**  | Apr 28, 2026 |
| Ben Henning | Android TL | **Tentative Go (OK to start HOW section)**  | Jan 16, 2026 |
| Sean Lip | Web TL | **Have Questions** | 27 Dec 2025 |
| Chris Skalnik | Reviewer for: [data handling and privacy](https://docs.google.com/document/d/1mnz8f708DZIa6BpUyRmbb0gCT6EKO3wIvWa_3rOEOYs/edit#bookmark=id.ij0czzwwz56s) | **Go (fully approved)**  | 8 Jul 2026 |

# 

# Overview

This TDD proposes a design for introducing Web-controlled Android feature flags and for serving Android-specific platform parameters via Oppia web backend. It introduces Android-specific storage models, evaluation handlers and an Android-admin dashboard that determine which flags and parameters are returned to the Android app based on app version, flavour, and rollout. This design establishes Web as the single source of truth for Android configuration while enabling safe staged rollouts and long-term compatibility across Android app versions.

# Problem Statement

Oppia’s Android app is approaching global availability (GA) and is actively evolving with new features, experiments and configuration changes. However, the current approach of managing Android feature availability and configuration is inflexible and unsafe for GA release.

**Android Configuration Issues**:  
	**Feature Release and Control Limitations**:

* Android feature flags are currently handled entirely within the Android codebase.  
* Once a feature is approved for release, the feature flag value is hardcoded to **true** before publishing the app to Play store.  
* Any change to feature behaviour or configuration requires shipping a new Android app version and waiting for users to update.

	**Absence of Android-specific Platform Parameters**:

* There is no Web-controlled system for managing Android-Specific configuration values such as update thresholds, supported API levels parameters.  
* Without remotely configurable platform parameters, forced upgrades, app deprecation and emergency configuration changes cannot be performed post-release.

	**Operational and safety Risks**:

* If a released Android version contains a critical bug or becomes unsupported, there is no way to remotely mark that version as deprecated.  
* Users on older or unsafe versions may continue using the app indefinitely, creating security, stability, and policy risks.  
* Feature rollouts cannot be staged or limited to specific environments (dev, alpha, beta, GA ) in a controlled manner.

**Impact on Global Availability** (GA):  
These limitations prevent Oppia Android from launching the Android app with app deprecation support, which is required capability for the initial public release.

This project aims to introduce a Web-controlled system for managing Android feature flags and Android-Specific platform parameters. This system will allow feature availability and configuration to be centrally managed, evaluated based on app version and release flavour, and delivered to the Android app at runtime.This will enable safer feature rollouts, app deprecation support, and long-term operational stability without requiring frequent Android app updates.

| Link to PRD (or N/A if there isn’t one) | N/A |
| :---- | :---- |
| **Target Audience** | Oppia Android developers (Release Coordinators) |
| **Core User Need** | As an Android release coordinator, I need a way to remotely configure Android feature flags based on app version and rollout rules, so that I can safely launch, stage, roll-back, or disable features without requiring users to update the app. |

# 

# **Section 1: WHAT**

*This section enumerates the requirements that the technical solution must satisfy. It will be used as a basis for “Section 2: HOW”.*

## Updates to Critical User Journeys

| Android Release Coordinator CUJs |  |  |  |  |
| ----- | ----- | ----- | ----- | ----- |
| **Setup for Android admin Dashboard CUJs** | Create three Users. LoggedIn User Web Release Coordinator Android Release Coordinator Log in as an Android Release Coordinator with the username “AndroidAdmin”. Log in as a Web Release Coordinator with the username “WebReleaseCoordinator”. Log in as a Logged-in user with the username “LoggedInUser”. Has two dummy feature flags, dummyff1, dummyff2 as LIVE and dummyff3 as FINAL. dummyff1: Min version: 11 Max version:23 Enabed: false Rollout percentage \= 0% dummyff2: Min version: 7 Max version:None Enabed: True Rollout percentage \= 0% dummyff3: Min version: 10 Max version:24. Has two dummy parameters: dummypp1:   default\_value: 10   rules:         min\_version: 11         max\_version: 21         app\_version\_flavour: dev           ~~alpha: true           beta: false           GA: false~~       value: 20 dummypp2:   default\_value: false   rules:         min\_version: 3         max\_version: 5         app\_version\_flavour:alpha       value: true  |  |  |  |
| **User type** | **Goal** | **Steps** | **Expectations** | **Mocks** |
| **(Regular) Logged-In user** | Fail to access Android Release coordinator page. | Type /android-release-coordinator into the URL bar. | \- User lands on the 404 error page. | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
| **Web Release Coordinator** | Access the Android Release coordinator page. | Type /android-release-coordinator into the URL bar. | \- User lands on the Android Release coordinator page dashboard. | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  | Access the Android Release coordinator page through the top right profile menu. | \- On clicking the top right profile menu. | \- Android Release coordinator page is shown in the list | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  |  | \- On clicking the Android Release coordinator page. | \- User lands on the Android Release coordinator page. |  |
|  | Failed to edit on Android Release coordinator page. | \- Try editing feature flags details for dummyff1. (min/max)/(on/off) | \- Not able to edit anything on the Android Release coordinator page. |  |
| **Android Release Coordinator** | Access the Android Release coordinator page | Type /android-release-coordinator into the URL bar. | \- User lands on the Android Release coordinator page. | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  | Access the Android Release coordinator page through the top right profile menu. | \- On clicking the top right profile menu. | \- Android Release coordinator page is shown in the list |  |
|  |  | \- On clicking the Android Release coordinator page. | \- User lands on the Android Release coordinator page |  |
|  | Can edit on Android Release coordinator page     (**Only LIVE feature flags**) | Set min\_version \> max\_version | \- Save disabled \- Error symbol shown beside save text on button. \- validation error shown on hovering over the buttonTooltip explains min ≤ max constraint |  |
|  |  | Set min\_version lower than hardcoded value | \- Save disabled \- Error symbol shown beside save text on button. \- validation error shown on hovering over the buttonTooltip explains min version restriction |  |
|  |  | \- Set valid min\_version and max\_version.\- Click Save button | \- Save enabled. \-  Changes persist and success toast is shown. |  |
|  | Fails to edit or interact with **FINAL feature flags**. | \- Attempt to interact with dummyff3. | \- All inputs are disabled; no edits are possible | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  | Configure **rollout percentage** | \- Set rollout percentage \= 50% on dummyff1 feature flag. | Save enabled. | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  |  | Click Save | Rollout persists. |  |
|  | View Android platform parameters | \- Open platform parameters from the navbar tab “**Platform Parameters**” | \- dummypp1 and dummypp2 are listed with default values | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  | Edit an existing Android platform parameter | \- Click on “Click to edit” button \- Change default value of dummypp1 from 10 → 15 \- Click **Save**. \- Enter **commit message** in confirmation modal \- Click the OK button. | \- Save button enabled,  \- Confirmation modal pop ups \- changes saved. \- Updated configuration is visible. | [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  | \- Click on “Click to edit” button \- Edit dummypp1 override value from 20 → 25\.\- Click **Save**. \- Enter **commit message** in confirmation modal \- Click the OK button. | \- Save button enabled,  \- Confirmation modal pop ups \- changes saved. \- Updated configuration is visible. |  |
|  |  | \- Click on “Click to edit” button \- Disable alpha for override of dummypp1. \- Click **Save**. \- Enter **commit message** in confirmation modal \- Click the OK button. | \- Save button enabled,  \- Confirmation modal pop ups \- changes saved. \- Updated configuration is visible. |  |
|  | Prevent invalid platform parameter updates | \- Click on “Click to edit” button \- Set min\_version \> max\_version | \- Save disabled \- Error symbol shown beside save text on button. \- validation error shown on hovering over the buttonTooltip explains min ≤ max constraint |  |
|  |  | \- Click on “Click to edit” button \- Set boolean value on dummypp1 | \- Save disabled \- Error symbol shown beside save text on button. \- validation error shown on hovering over the button |  |

## Other product requirements that are not captured by the CUJs

**Addition of a new user role**: Android Release Coordinator and **Renaming Release Coordinator role to Web Release Coordinator:**  
Should refer this wiki: [https://github.com/oppia/oppia/wiki/Instructions-for-editing-roles-or-actions\#adding-a-new-role](https://github.com/oppia/oppia/wiki/Instructions-for-editing-roles-or-actions#adding-a-new-role)

Rename all the current Release Coordinator files to Web Release Coordinator and make   
appropriate changes  
.  
**Refactor “core/controllers/release\_coordinator.py” and make it explicitly “Web Release Coordinator”–scoped**

- Rename file to controllers/web\_release\_coordinator.py  
- Rename handler classes  
  - MemoryCacheHandler → WebMemoryCacheHandler  
  - FeatureFlagsHandler → WebFeatureFlagsHandler  
  - FeatureFlagsHandlerNormalizedPayloadDict → WebFeatureFlagsHandlerNormalizedPayloadDict  
- Will mirror this for controllers/android\_release\_coordinator.py  
- Rename to make web specific @acl\_decorators.can\_access\_release\_coordinator\_page  
- Will mirror this decorator too for android.


  
**Migration of existing Android feature flags and platform parameters**:

* Platform Parameter:  
  * [https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/data/src/main/java/org/oppia/android/data/backends/gae/api/PlatformParameterService.kt\#L21](https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/data/src/main/java/org/oppia/android/data/backends/gae/api/PlatformParameterService.kt#L21)  
  * [https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/utility/src/main/java/org/oppia/android/util/platformparameter/PlatformParameterConstants.kt\#L166](https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/utility/src/main/java/org/oppia/android/util/platformparameter/PlatformParameterConstants.kt#L166)  
* Feature flags:  
  * [https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/utility/src/main/java/org/oppia/android/util/platformparameter/FeatureFlagConstants.kt](https://github.com/oppia/oppia-android/blob/f9106d91297abd09b24da25d5485deb8473b0125/utility/src/main/java/org/oppia/android/util/platformparameter/FeatureFlagConstants.kt)  
* Android Wiki:  
  * [https://github.com/oppia/oppia-android/wiki/Platform-Parameters-%26-Feature-Flags](https://github.com/oppia/oppia-android/wiki/Platform-Parameters-%26-Feature-Flags)

    

    

    

**Android Feature flags**

1. Classification of Android feature flags:  
   1. **LIVE**: Flags that can be toggled dynamically to support staged rollouts, experimentation, and rollback.  
   2. **FINAL**: Flags whose values are permanently fixed and can no longer be changed.

Note: We need to make a PR for moving Feature flags to the FINAL stage. A LIVE flag with 100% rollout is still configurable; it can be rolled back, disabled, or have its version constraints updated. This is useful during rollout stabilization or when monitoring post-launch behavior.

FINAL, on the other hand, represents a lifecycle transition where the feature is considered fully launched and no longer needs dynamic control. At this stage, the flag becomes immutable and is effectively treated as part of the baseline app behavior. So the transition from LIVE → FINAL is a decision made once we are confident the feature is stable and no longer requires rollout control, rather than being automatically tied to rollout\_percentage reaching 100\.

2.  Exposure of feature flags by Android flavour :   
   

| Android Flavours | LIVE feature flags | FINAL feature flags |
| ----- | :---: | :---: |
| dev | Not exposed | Exposed |
| alpha | Not exposed | Exposed |
| beta | Not exposed | Exposed |
| GA | Exposed | Exposed |

   

* For the **dev**, **alpha**, and **beta** flavours, **LIVE feature flags** must be surfaced only via the **Android dev options menu**, where app users( android dev) can manually control their values.  
  **Note:** All feature changes require an app restart (and the built-in dashboard in the app enforces this) since the app snapshots all platform and flag states on startup to avoid potential instability or inconsistency issues.  
* For the **GA** flavour, both **LIVE and FINAL feature flags** must be sourced exclusively from the **Web backend**.  
    
3. Feature flag evaluation semantics:

   The Android app must interpret the Web response for feature flags according to the following rules:

   1. If a feature flag is **absent** from the Web response, the Android app determines the flag’s value using its local defaults or dev options.  
      2. If a feature flag is **present** in the Web response with a **value of OFF**, this explicitly indicates that the **server is disabling the feature**.  
4. Android feature flags stay in the dashboard forever (Just shifts from LIVE to FINAL list).  
5. Each Android feature flag needs a **min version** and a **max version**. The min \+ max versions should be hardcoded in the codebase, but can be overwritten from Android-Admin dashboard.  
   1. However, the following restrictions must be adhered to:  
      1. Min version cannot be earlier than the one that's hardcoded.  
      2. Max version ~~has no restriction~~ if provided, needs to be greater than or equal to min version.  
      3. Min must be less than or equal to max.  
   2. If the **app version falls outside \[min, max\],** the feature flag should be **completely omitted from the Web handler output** (instead of being returned and defaulted to false/true).

**Platform Parameters**:

1. Introduction of a new controller endpoint similar to [FeatureFlagsEvaluationHandler](https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/controllers/feature_flag.py#L28) that:  
   1. Evaluates the list of platform parameters specific to Android, and filters based on the provided Android version name and flavour (provided via GET).  
   2. Returns a response of a list of the parameters as objects with name and values.  
2. Notes on Evaluation Matching:  
   1. The current environment must be used when determining parameter applicability across **dev**, **alpha**, **beta**, and **GA** stages.  
   2. Only Android-specific platform parameters must be evaluated and returned by this endpoint.  
   3. The supplied app version name must be matched against configured parameter conditions, specifically using the version and flavour components of the version string.  
3. Each Android platform parameter follows the same rule-based structure as existing Web platform parameters.  
   1. Parameter consists of:  
      1. “default value”  
      2. “list of conditional rules”  
   2. Each rule specifies the condition under which it applies.  
   3. When evaluating a parameter, rules are evaluated in order, and the first matching rule is selected. If no rule matches, the default value is used.  
      For reference: [https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py\#L344](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py#L344)  
      → Refer **CUJs dummypp1** structure for more info.

## Technical Requirements

### Additions/Changes to the Web Server Interface

| \# | Endpoint URL | Request type (GET, POST, etc.) | New / Existing | Description of the request/response contract (and, if relevant, how it’s different from the previous one). For new endpoints, include which [access control decorator](https://github.com/oppia/oppia/blob/develop/core/controllers/acl_decorators.py) (e.g. “can\_play\_exploration”, “open\_access”, etc.) will be used. Notes: Highlight new endpoints/fields in green and deleted endpoints/fields in red. Include “bookmark” links to later sections of the doc that provide the relevant implementation details.[^1] |
| :---- | :---- | :---- | :---- | :---- |
| 1\. | /android\_feature\_flagsAndroidFeatureFlagsHandlerPassed via header : app\_version\_name, installation\_id, app\_package\_name  | GET | Existing(extended) | Returns evaluated Android feature flags applicable to given app version and installation ID.The response is a list of {name , enabled} objects.Feature flags are filtered by min/max version, rollout percentage, and Android user groups.If a flag is outside the applicable version range, it is omitted from response.Uses “**open\_access**”. |
| 2\. | /android\_platform\_parametersAndroidPlatformParametersHandler Passed via header : app\_version\_name, installation\_id app\_package\_name | GET | Existing(extended) | Returns evaluated Android platform parameters applicable to given app version.The response is a list of {name, value} objects. Parameters are resolved using default values and version/flavour-specific overrides. Uses “**open\_access**”. |
| 3\. | **/can\_access\_web\_release\_coordinator\_page/can\_access\_release\_coordinator\_page**WebReleaseCoordinatorAccessValidationHandler | GET | NEW | Renamed the old handler “[ReleaseCoordinatorAccessValidationHandler](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/access_validators.py#L372)” Validates access to Web Release Coordinator page. Uses a renamed decorator(can\_access\_release\_coordinator\_page) “**can\_access\_web\_release\_coordinator\_page**” |
| 4\. | **/**can\_access\_android\_release\_coordinator\_pageAndroidReleaseCoordinatorAccessValidationHandler | GET | NEW | Mirrors “WebReleaseCoordinatorAccessValidationHandler” Validates access to Android Release Coordinator page. Uses a new decorator “**can\_access\_android\_release\_coordinator\_page**” |
| 5\. | **/**web\_release\_coordinator/feature\_flags/feature\_flagsWebFeatureFlagsHandler | GET/PUT | NEW | Renamed the old handler“[FeatureFlagsHandler](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/release_coordinator.py#L144)” Handler for web feature flags.  |
| 6\. | /android\_release\_coordinator/feature\_flagsAndroidFeatureFlagsAdminHandler | GET/PUT | NEW | Mirrors “WebFeatureFlagsHandler”Handler for android feature flags.**GET** Returns following arguments: feature\_name feature\_state (LIVE/FINAL) min\_version max\_version rollout\_percentage **PUT** Updates an existing Android feature flag min\_version max\_version rollout\_percentage Uses a new decorator “**can\_access\_android\_release\_coordinator\_page**”  |
| 8\. | /android\_release\_coordinator/platform\_parameters AndroidPlatformParametersAdminHandler  | GET/PUT | NEW | Handler for android platform parameters.Reference: [AdminHandler](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py#L254)**GET** platform\_param\_name default\_value List of Rules min\_app\_version max\_app\_version List of flavours(dev, alpha, beta, GA) value **PUT** platform\_param\_name default \_value rules commit\_message Uses a new decorator “**can\_access\_android\_release\_coordinator\_page**” |

### New/Changed Calls by the Web Frontend / Android Client to the Web Server Interface

| \# | Endpoint URL | Request type (GET, POST, etc.) | Description of why the new call is needed, or why the changes to an existing call are needed |
| :---- | :---- | :---- | :---- |
| 1\. | /android\_feature\_flagsPassed via header : app\_version\_name, installation\_id, app\_package\_name | GET | Called by Android app to determine which Android feature flags should  be enabled for current app version and device.Enables staged rollouts, experimentation, and app deprecation support. |
| 2\. | /android\_platform\_parametersPassed via header : app\_version\_name,  app\_package\_name | GET | Called by Android app during startup and periodic sync to fetch Android-specific configuration values required for runtime tuning, safety checks without requiring a new app release. |
| 3\. | **/**can\_access\_web\_release\_coordinator\_page | GET | Called by Web frontend to validate if the user has access to web release coordinator page. |
| 4\. | /can\_access\_android\_release\_coordinator\_page | GET | Called by Web frontend to validate if the user has access to android release coordinator page. |
| 5\. | **/**web\_release\_coordinator/feature\_flags | GET / PUT | Called by the Web Release Coordinator page for managing Web feature flags. |
| 6\. | /android\_release\_coordinator/feature\_flags | GET / PUT | Called by the Android Release Coordinator page for managing Android feature flags. |
| 8\. | /android\_release\_coordinator/platform\_parameters | GET/PUT | Called by Android Release Coordinator page for managing **Android Platform Parameters.** |

### From Ben: The rough expectation is that the **app calls the endpoint no more often than 12 hours** ([https://github.com/oppia/oppia-android/blob/4bda09fd03b860ac50143c51e8c40ccf6965dcde/config/src/java/org/oppia/android/config/platform/platform\_parameters.textproto\#L10](https://github.com/oppia/oppia-android/blob/4bda09fd03b860ac50143c51e8c40ccf6965dcde/config/src/java/org/oppia/android/config/platform/platform_parameters.textproto#L10)).

### We use something called WorkManager to manage these jobs, and they will run at most as often as we ask (so in this case, it will try to run every 12 hours but may sometimes not run if the device is constrained or not connected to the internet). This is an important distinction. Our currently 1DAU is about 400, but the total fleet is about 23k. Now it won't be the case that all of those users will get the version of the app that properly enables synchronizing at the same time, but it will grow that over a few months. Worst case is 23k current user devices synchronizing every 12 hours which comes out to about 0.53QPS. That's actually quite a bit and something I will bring up with the web TLs. (/cc @a0raca1p@gmail.com and @kevintab95@gmail.com). Note that we expect a significant audience increase from GA marketing this year, and our peak installs to date was 45k which would be \~1.04QPS load on the endpoint.

### One note: work manager won't let us synchronize more often than every 15 minutes (so even though this is a configurable parameter there's no way for Oppia web to accidentally DDoS itself, though a 15 minute synchronization is 48x increase which turns our peak into 50QPS which would probably not be great. Fortunately it can't be worse than that.

### Separately, when auditing this I noticed in passing that it will become possible for Oppia web to basically permanently break syncing on Android and subsequently filed [https://github.com/oppia/oppia-android/issues/6097](https://github.com/oppia/oppia-android/issues/6097) to put a guardrail in place on the app side to prevent this (the idea is to ignore any value web gives the app that exceeds a 1 month synchronization interval).  One follow-up: synchronization will never happen when there isn't connectivity, but certain events like opening the app or connecting to the internet can trigger synchronization to start ('can' is key here since work manager cannot make actually guarantees).  One more follow-up: QPS estimates above should be 2x'd because the app will synchronize feature flags and platform parameters independently.  Web Server Storage Model Additions/Changes

| \# | Datastore model | Description of changes | How will existing data be handled? |
| :---- | :---- | :---- | :---- |
| 1\. | BaseFeatureFlagConfigModel | \- New abstract base model to hold shared fields and validation logic for web and Android feature flags. | \- Existing Web data remains valid. \- validation ensures unused fields are set to None. |
| 2\. | BasePlatformParameterConfigModel | \- New abstract base model to hold shared fields and validation logic for web and Android platform parameters. | \- Existing Web data remains valid. \- validation ensures unused fields are set to None. |
| 3\. | WebFeatureFlagConfigModel | \- Existing [Web feature flag](https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae_models.py#L137) storage (FeatureFlagConfigModel) will be renamed to WebFeatureFlagConfigModel. \- This Datastore model will be treated as Web-only feature flag storage model.  | \- Existing data remains unchanged and continues to be served to Web only. |
| 4\. | WebPlatformParameterConfigModel | \- Existing [platform parameter](https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae_models.py#L56) storage will be renamed to WebPlatformParameterConfigModel \-This Datastore model will be treated as web-only platform parameters storage model.  | \- Existing data remains unchanged and continues to be served to Web only. |
| 5\. | AndroidFeatureFlagConfigModel | \- New model to store Android-specific feature flags, including  feature\_state (LIVE/FINAL) min\_app\_version max\_app\_version "Rollout\_percentage"  | \- New model; no migration required.\- The value for the "'rollout\_percentage'" by default will be 0, \- min\_version will have a default min value \- max\_version will not be defined |
| 6\. | AndroidPlatformParameterConfigModel | \- New model to store Android-specific platform parameters, including Default values  Rules (list) min\_app\_version max\_app\_version app\_version\_flavour Value | \- New model; no migration required. |

### Data Handling and Privacy

| \# | Type of data | Description | Why do we need to store this data? | Anonymized? | Can the user opt out? | Wipeout policy | Takeout policy |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| ~~1\.~~ | ~~Android User-groups and the users tied to it.~~ | ~~Set of installation ID that fall into the user\-group, for them the feature will be enabled~~ | ~~This allows us to keep track of the installation ID for which the feature should be enabled.~~ | ~~No; The data related to the users will be visible to admins, i.e; the installation ID which are part of user-groups will be visible to admins.~~ | ~~This property is for the use of admins only, in a way this does not affect the end users because here mainly we want to enable the feature for some set of users~~ | ~~Remove the Installation ID from the group when the user uninstalls the app.~~ | ~~We will include the names of the user-groups that the installation ID is part of when they want to download their data.~~ |
| 1\. | Percentage rollouts for Android users. | The percentage rollout set by the Android release coordinator for the users. Here the feature flag will be enabled for the defined percentage of the Android users. | This allows us to decide what percentage of users will see an enabled feature flag. | N/A | This feature is for the use of  android release-coordinators only, in a way this does not concern the end users because here mainly we want to check that the feature-flag we have which is in the GA stage works fine for the end users. | N/A; data is not tied to a user. | N/A; data is not tied to a user |

### Connection to Existing Work

1. Found Hitesh’s TDD : [TDD -- Enable Percentage Rollout](https://docs.google.com/document/d/1NZqkP2uCVEmEU80sIP7_zswATGZstPdMzDvaIn8QK-k/edit?tab=t.0) (Kind of similar at few points)  
2. **Addition of a new user role and renaming Release coordinator role to Web Release Coordinator**: Android Release Coordinator.  
   Should refer this wiki: [https://github.com/oppia/oppia/wiki/Instructions-for-editing-roles-or-actions\#adding-a-new-role](https://github.com/oppia/oppia/wiki/Instructions-for-editing-roles-or-actions#adding-a-new-role)  
   ![][image1]  
3. Reference for Storage layer models. [https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae\_models.py](https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae_models.py)  
4. Reference for platform parameter:  
   [https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py\#L344](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py#L344)  
5. All the UI components are taken from release coordinator and admin page. [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) see this mock for more details.

### Other Requirements

It's important to compute the on/off status of all the feature flags and configurations of platform parameters quickly and efficiently, since android would be fetching their state on pretty much every call to the server. Otherwise all the user journeys would become quite slow.

## Remaining Open Questions

1. Can you tell me how does android release a new feature currently, like they do like oppia web, i.e., dropping the feature flag from the codebase or do they just hardcode the feature flag value to yes?  
   * **Answer**: Android currently hardcodes the feature flag value to yes as part of their release process.  
2. How do they currently handle feature flags specific to a version and platform parameters?  
   * **Answer**:  As above, they hardcode it before releasing. This is quite inflexible.  
3. Why is this currently a blocker for GA release?  
   * **Answer**: We need to be able to set a flag to say "current app is deprecated, please upgrade". This flag needs to be switched on remotely, it shouldn't be done when the app is deployed on Play Store (otherwise no one can use it).  
4. Can you tell me What android Rcs can do from Android admin dashboard?, asking for CUJs  
   * This is what I know,  
     1.  They can add LIVE feature flags, with min version  
     2.  They can see all the LIVE and FINAL flags  
     3. They can edit min/max version for them, turn on/off LIVE flags, rollout  
     4.  They can see the platform parameter and default values for them, Can they edit it and add them too?  
     5.  Can we move feature flags from LIVE to FINAL from this dashboard?  
   * **Answer**:  
     1\. They can't do this directly. The feature will need to be added in code and then the RCs can turn it to LIVE / FINAL / etc.  
     4\. They can edit but not add.  
     5\. Yes, I think so.  
5. Is FeatureFlagConfigModel the correct storage model?  
   * **Answer**: yup I think FeatureFlagConfigModel is correct; more generally the idea is to just separate any current "feature flag" things into "web feature flag" and "android feature flag".  
6. Is my way of storing Platform parameters okay or any other suggestion ?  
   My way : “Each Android platform parameter consists of a default value and an optional list of conditional overrides. Each override specifies the conditions under which it applies (such as minimum/maximum app version and Android flavour) and the value to use when those conditions are met. When evaluating a parameter, the Web backend selects the most specific matching override; if none match, the default value is used.”  
    	Example: dummypp :  
* Default Value: false  
* Type: Boolean  
* Override :   
  \[

    {

      min\_version: null,

      max\_version: null,

      flavours: {

        dev: true,

        alpha: true,

        beta: false,

        GA: false

      },

      value: true

    }

  \]  
  **Answer**: see how the platform params works in /admin currently \-- I just meant following that same system. Ie if multiple filters match we pick the first one, like in the rules / feedback of the exp editor.  
  Refer this link: [https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py\#L344](https://github.com/oppia/oppia/blob/051712804a4cbb453ca1f0272d503ebb881bc6a8/core/controllers/admin.py#L344)

7. Should we create the same system for adding an Android feature flag like in Oppia Web?  
   Link for reference:   
   * [https://github.com/oppia/oppia/wiki/Launching-new-features\#follow-the-steps-below-to-add-a-new-feature-flag](https://github.com/oppia/oppia/wiki/Launching-new-features#follow-the-steps-below-to-add-a-new-feature-flag)

		**Answer**:Yes.

8. Should we create a separate list and registry for android platform parameters, like the web?  
   Link for reference:   
   * [https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform\_parameter\_list.py](https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform_parameter_list.py)  
   * [https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform\_parameter\_registry.py](https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform_parameter_registry.py)  
     **Answe**r:Yes.

| STOP\! *Please get at least a “tentative go” decision from the tech leads before continuing to the HOW section. If you write out a full spec below, this work might be wasted if we decide not to proceed with the proposed feature/change or the goals/problem statement needs adjusting. Reviewers will let you know whether to carry on, or what additional changes are needed to do so (e.g. adding a PRD). Note: A “tentative go” decision at this point just means that we’re committing to fleshing out the HOW section. A “tentative go” or “go” decision can change to a “no-go” later on for various reasons: lack of feasibility, too complicated, not enough resources, change of strategic direction, etc.* |
| ----- |

## Reviewers will verify:

* Should we tackle this problem at this time?  
* Requirements:  
  * If the project requires a PRD, is one linked to this document, and is the PRD sufficiently ready for technical design to proceed?   
  * Do the requirements in this TDD match those in the PRD?  
* Can the project be done using only existing patterns in the codebase? Are there any approaches/insights that might be useful for designing the solution?   
* Are all assertions properly justified (with links to sources/proof, if appropriate)?  
* Does the testing plan validate all required user stories in the product spec?

# Section 1 Review Result

| *The Android/Web TL should complete this section.* |
| :---- |

| Decision | Go | No-Go |
| :---- | :---- |
| **Rationale (if no-go)** |  |
| **Technical design constraints, if applicable (and rationale for each)** | *Are there any additional requirements that need to be imposed (from a technical perspective)? If so, list them here.* |
|  | … ... |
| **Product team partner (if applicable)** |  |
| **Note to Reviewers*:*** *If you give a “Go” decision, please also delete any remaining yellow boxes in Section 1, as well as the “Reviewers will verify” section above this one. This will mark the WHAT part as “approved” and make that section of the TDD easier to read.* |  |

---

# **Section 2: HOW**

## Existing Status Quo

Currently, Oppia Web supports **Web feature flags** and **Web platform parameters** that are managed via the **Release coordinator** and **Admin UI** and stored in the datastore.These are primarily used to control web-specific behaviour.  
Storage model : [Web feature flag](https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae_models.py#L137),  [platform parameter](https://github.com/oppia/oppia/blob/f6bfac67e2e1aa5ebfadb36c3622e62e1f708053/core/storage/config/gae_models.py#L56)

On the Android side, **feature flags and platform parameters** are declared as **constants** with fixed default values inside the codebase.  
Refer below files : [FeatureFlagConstants.kt](https://github.com/oppia/oppia-android/blob/fccae19960d3ea99b9f75e6880c73c75b8af480a/utility/src/main/java/org/oppia/android/util/platformparameter/FeatureFlagConstants.kt)  ,[PlarformParameterConstants.kt](https://github.com/oppia/oppia-android/blob/37cb257e9281aa317f28f1eaddf70804c6054b39/utility/src/main/java/org/oppia/android/util/platformparameter/PlatformParameterConstants.kt)

This approach introduces several limitations:  
 1\. Any change to feature flag or platform parameter requires a code change followed by a new Android app release.  
2\. Feature flags cannot be dynamically overridden based on app version and release flavour or  gradual rollout strategies.

As a result, Android feature configuration is less flexible compared to the existing Web feature flag and Platform parameter system.

## Solution Overview

This proposal introduces a Remote Configuration system for Android Feature Flags & Platform Parameters, managed through the Oppia Web Admin interface and served to Android clients at runtime.

The solution follows the **existing Oppia architectural patterns** used for web feature flags and Platform parameters.This design will separate Web-only and Android-only configuration, and introduces shared base models to centralize common structure (BasePlatformParameterConfigModel, BaseFeatureFlagConfigModel).

Create a similar system of **registry for android platform parameters** as used for web platform parameters.  
For reference: 

* [https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform\_parameter\_list.py](https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform_parameter_list.py)  
  * [https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform\_parameter\_registry.py](https://github.com/oppia/oppia/blob/d221cfc7603e200494aec4b200c3230a86bf2ae5/core/domain/platform_parameter_registry.py)

For **adding new android feature flags** we will create a similar system as used by web feature flags.  
For reference:

* [https://github.com/oppia/oppia/wiki/Launching-new-features\#follow-the-steps-below-to-add-a-new-feature-flag](https://github.com/oppia/oppia/wiki/Launching-new-features#follow-the-steps-below-to-add-a-new-feature-flag)

**1\.  Base Models:**  
![][image2]

* Two new abstract base models are created in this file: [core/storage/config/gae\_models.py](https://github.com/oppia/oppia/blob/1c04360467ab3935d199f02010e3bfce9a4ce819/core/storage/config/gae_models.py) :  
  * **BaseFeatureFlagConfigModel**  
    * Defines shared structure for feature flag configuration across web and Android.  
      * **Id →** Feature flag name  
      * **created\_on** → Inherited from [BaseMode](https://github.com/oppia/oppia/blob/1c04360467ab3935d199f02010e3bfce9a4ce819/core/storage/base_model/gae_models.py#L160)l  
      * **last\_updated** → Inherited from BaseModel  
      * **deleted** → Inherited from BaseModel  
  * **BasePlatformParameterConfigModel**  
    * Defines shared structure for platform parameters across web and Android.  
      * **Id →** Platform parameter name  
      * **created\_on** → Inherited from [VersionedModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/base_model/gae_models.py#L866)  
      * **last\_updated** → Inherited from VersionedModel  
      * **deleted** → Inherited from VersionedModel  
    * Snapshot models ([PlatformParameterSnapshotMetadataModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/config/gae_models.py#L37), [PlatformParameterSnapshotContentModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/config/gae_models.py#L45)) remain unchanged and are reused.  
    * **Versioning & Snapshots:**  
* **BasePlatformParameterConfigModel** extends [VersionedModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/base_model/gae_models.py#L866).  
* Therefore, both **Web and Android platform parameters** are versioned.  
* Existing snapshot models are reused:  
  * [PlatformParameterSnapshotMetadataModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/config/gae_models.py#L37)  
  * [PlatformParameterSnapshotContentModel](https://github.com/oppia/oppia/blob/d79ab43c96a8681859fedceb775d25f0a9bb3602/core/storage/config/gae_models.py#L45)  
* No new snapshot models are introduced.

**2\. WebFeatureFlagConfigModel:**  
	**Extends: BaseFeatureFlagConfigModel**  
![][image3]  
**Fields**:

* force\_enable\_for\_all\_users  
* rollout\_percentage  
* user\_group\_ids

**Notes**`:`

* Renamed from existing FeatureFlagConfigModel  
* All the related services and domain files and functions will be renamed to specify it serves Web-only feature flags (prefix “web”).  
* No migration required.

**3\. AndroidFeatureFlagConfigModel:**  
	**Extends: BaseFeatureFlagConfigModel**  
![][image4]  
**Fields**:

* **state**   
  * **LIVE** : The feature flag is configurable and can be updated.  
  * **FINAL**: The feature flag is locked and immutable.  
* **min\_app\_version**  
* **max\_app\_version**  
* **rollout\_percentage** 


**Notes**`:`

* All the related services and domain files and functions will be named with prefix “android”

**Domain Layer Validations**:

* Android feature flag domain layer enforces the following validations to prevent invalid data from being persisted:  
  * Feature flags in **FINAL** state cannot be updated  
  * **rollout\_percentage** must be an integer between 0 and 100\.  
  * **min\_app\_version** must be defined, and it can’t be smaller than default value.  
  * **max\_app\_version** (if defined) must be \>= min\_app\_version  
  * Android feature flags must not contain web-only fields.(see this for more detail: [TDD - Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1OyT2jfL-2K3gZ3weWLQhRtjxqfwkfcADM3I1xAObFas/edit?tab=t.0#bookmark=id.3ur674kvpl82))

**The complete solution flow will be as follows:**

* **Step 1: Model Creation:**  
  * Create a new **AndroidFeatureFlagConfigModel** with the fields:  
    * feature\_state  
    * min\_app\_version  
    * Max\_app\_version  
    * Rollout\_percentage  
* **Step 2: Domain Layer**:  
  * Create **android\_feature\_flag\_domain.py,** which defines:  
    * validate()  
    * from\_dict()  
    * to\_dict()  
    * serialize()  
    * deserialize()  
  * This layer contains all validation and evaluation logic.  
* **Step 3: Service Layer**:  
  * Create **android\_feature\_flag\_service.py,** which exposes service-level functions for:  
    * Fetching Android feature flags.  
    * Updating Android feature flags.  
  * These functions are invoked by controller handlers.  
* **Step 4: Registry:**  
  * Create **android\_feature\_flag\_registry.py**, responsible for:  
    * Registering all Android feature flags.  
    * Providing lookup utilities.  
    * Acting as the source of truth for valid feature flag names.  
* **Step 5: Controllers:**  
  * Introduce Android-specific handlers under:  
    * /android-release-coordinator/feature-flags (accessible only to Android Release coordinators)  
    * /android-feature-flags (for Android clients)  
  * These handlers:  
    * Accept Android-specific requests.  
    * Return only Android feature flag data.  
* **Step 6: Tests :**   
  * Tests will be added for:  
    * Domain Validation  
    * Rollout evaluation logic  
    * Handler permissions and responses.

**Feature Flag Evaluation Logic :**   
Feature flag evaluation is performed at request time using the **Android Installation ID.**

- **Rollout Percentage Evaluation:**

	The rollout mechanism determines whether a feature flag is enabled for a given Android installation in a **deterministic and stateless manner.**

- **Key Design Principles:**  
* The installation ID is treated as a **reasonably unique ID string**, not tied to any specific implementation, so that:  
  * The client can change its ID generation strategy in the future without breaking rollout logic.  
  * Rollout decisions must be:  
    * Deterministic (same input → same output)  
    * Independent across feature flags.  
    * Uniformly distributed  
    * Stateless and recomputable.

- **Salting Strategy:**  
* To prevent the same installation from being bucketed identically across all features, **the installation IDs are salted using the feature flag name (**as Feature flag names are unique**).**  
* **This ensures:**  
  * Different feature flags have independent rollout distributions.  
  * No correlation between different feature rollouts for the same device.  
- **Rollout Computation Algorithm:**


| unique\_feature\_install\_id \= f"{installation\_id}:{feature\_flag\_name}".encode('utf-8') sha1\_hash \= hashlib.sha1(unique\_feature\_install\_id) sha1\_hex \= sha1\_hash.hexdigest() candidate\_id \= int(sha1\_hex\[:8\], 16\) is\_enabled \= (candidate\_id % 100\) \< rollout\_percentage |
| :---- |


- **App Version Filtering:**  
* Before rollout evaluation:  
  * The Android app version is checked against **min\_app\_version** and **max\_app\_version**.  
  * If the **version is outside the range**, the feature flag is absent in the web response to Android (It means Android app decides the flag’s value).

**4\. WebPlatformParameterConfigModel:**  
	**Extends: BasePlatformParameterConfigModel**

![][image5]  
**Fields:**

* rules  
* rule\_schema\_version  
* Default\_value

**Notes:**

* This corresponds to the existing [PlatformParameterModel](https://github.com/oppia/oppia/blob/34f783ffe2f05429f5d5ded1427c765f66346780/core/storage/config/gae_models.py#L56).  
* All the related services and domain files and functions will be renamed to specify it serves Web-only platform parameters (prefix “web”).  
* No migration required.

**5\. AndroidPlatformParameterConfigModel:**  
	**Extends: BasePlatformParameterConfigModel**  
![][image6]

**Fields:**

* **rules:** list(dict). List of dict representation of

                AndroidPlatformParameterRule objects, which have the following  
                structure:  
                    \- **value\_when\_matched:**  The result of the rule when it's matched.  
                    \- **filters**: list(dict). List of dict representation of AndroidPlatformParameterFilter objects, having the following structure:  
                            \- **type**: str. The type of the filter.  
                            \- **conditions**: list((str, str)). Each element of the  
                                list is a 2-tuple (**op, value**), where op is the  
                                operator for comparison and value is the value  
                                used for comparison.

* **rule\_schema\_version:** The schema version for the rule dicts.  
* **default\_value:** The default value of the platform parameter

**Notes:**

* All the related services and domain files and functions will be named to specify it serves Android-only platform parameters (prefix “android”).

**Domain Layer Validations:**  
The Android platform parameter domain layer enforces the following validations to ensure the following validations to ensure correctness and prevent invalid data from being persisted:

* Each Android platform parameter must have a **default value**.  
* Data type consistency:  
  * default\_value and value\_when\_matched must match the declared data type.  
* Rule structure  
  * Each rule must contain:  
    * Valid list of filters  
    * value\_when\_matched  
* Filter validation:  
  * Only supported filter types are allowed:  
    * app\_min\_version  
    * app\_max\_version  
    * flavour  
  * Operators must be valid for the given filter type.  
* Validations fail if web-specific configuration is present.

**The complete solution flow will be as follows:**

* **Step 1: Model Creation:**  
  * Create a new **AndroidPlatform ParameterConfigModel** with the fields:  
    * **rules**  
    * **rule\_schema\_version:**   
    * **default\_value**  
  * This model stores only **Android-specific platform parameters.**  
* **Step 2: Domain Layer:**  
  * Create **android\_platform\_parameter\_domain.py,** which defines:  
    * validate()  
    * from\_dict()  
    * to\_dict()  
    * serialize()  
    * deserialize()  
    * evaluate(context)  
* **Step 3: Service Layer**:  
  * Create **android\_platform\_parameter\_service.py,** which exposes service-level functions for:  
    * Fetching all Android platform parameters.  
    * Evaluate parameters for a given Android request.  
    * Return resolved parameter name/value pairs..  
  * These functions are invoked by controller handlers.  
* **Step 4: Registry:**  
  * Create **android\_platform\_parameter\_registry.py**, responsible for:  
    * Registering all Android platform parameters.  
    * Providing lookup utilities.  
    * Acting as the source of truth for valid Android platform parameter names.  
* **Step 5: Controllers:**  
  * Introduce Android-specific handlers under:  
    * /android-release-coordinator/platform-parameters (accessible only to Android Release coordinators)  
    * /android-platform-parameters (for Android clients)  
  * These handlers:  
    * Accept Android-specific requests. (containing the required headers like app\_version\_name and app\_package\_name for evaluation)  
      Note: Currently, these endpoints are designed as open-access (similar to existing feature flag and platform parameter endpoints), and we rely on request headers like app\_version\_name and app\_package\_name for evaluation.Here from Android-specific i meant, having the required headers present and this can be of course spoofed, but it does not matter, as we only send configuration data, no user-related data.  
    * Return evaluated Android platform parameter values.  
* **Step 6: Tests :**   
  * Tests will be added for:  
    * Domain Validation  
    * Rule evaluation order.  
    * Handler permissions and responses.

**Platform Parameter Evaluation Logic :**   
Platform parameters evaluation is performed at request time using the **app version.**

- **Example of app-version :**   
  - **12.5-5f58b8fbeb-dev' is the app version name.** Specifically:  
  - '12' is the major version.  
  - '5' is the minor version.  
  - '5f58b8fbeb' is the commit from which the release candidate was built.  
  - 'dev' is the configuration (flavor) of the app built.  
- **Rule Evaluation Order:**  
* Rules are evaluated in the order they are defined.  
* The first matching rule determines the parameter value.  
* If no rule matches:  
  * The default\_value is returned.

- **App Version & flavour matching:**  
* The app version string from the request header is parsed into:  
  * Version number (e.g. `12.5`)  
  * flavour (e.g. `dev`)  
* Each rule’s filters are checked against:  
  * The parsed version number  
  * The parsed flavour  
* A rule is considered a match only if all its filters evaluate to true.  
* If the Android app version falls outside all rule conditions, the default value is used.

## Key High-Level and Architectural Decisions

### Decision 1: How should Android feature flags be modeled in storage?

**Problem Statement:**  
Android feature flags have different lifecycle rules (LIVE/FINAL, min\_app\_version, max\_app\_version) than web feature flags.

We have considered the following alternatives:

1. **Extend existing FeatureFlagConfigModel**  
2. **Create a separate AndroidFeatureFlagConfigModel**

Among these, we believe that second is the best approach, because:

* Android feature flags have different lifecycle rules(LIVE/FINAL, min\_app\_version, max\_app\_version) from web feature flags.  
* Some of the properties like, **force\_enable\_for\_all\_users, user\_group\_ids** are not relevant to android feature flags but need to be present for web feature-flags.  
* In the future if we plan to have some properties that need to be associated with android-feature-flags or web-feature flags, we can easily update the relevant models to achieve it.

The above approaches are contrasted in detail in the following table:

|  | Existing Model | New Model |
| :---- | :---- | :---- |
| Code & system maintainability and readability | Making changes to the existing **FeatureFlagConfigModel** will mix-up the properties of both android feature-flags and web feature flags. Properties for web feature flags  are: name description feature\_stage rollout\_percentage force\_enable\_for\_all\_users user\_group\_ids Properties for android feature-flags: name description feature\_state min\_app\_version max\_app\_version rollout\_percentage There are some properties that are unique to web feature-flags and android feature flags, like: Unique to web feature-flags: feature\_stage user\_group\_ids force\_enable\_for\_all\_users Unique to android feature flags: feature\_state min\_app\_version max\_app\_version This will in the end result in reducing both the readability and maintenance. In future if there are some properties that are unique to web feature-flags and android feature flags, we would require to add extra comments, validations to make sure that the properties are correctly assigned. | With the help of a new model we will be able to separate the code for android feature-flags and web feature flags. It will be a lot easier to maintain the functionalities of android feature-flags and web-feature flags. The readability will automatically increase. In future if there are some properties that are unique to android feature-flags and web feature flags, it will be easier to implement. |
| Consistency with existing codebase | We will be able to use the existing code and structure to implement the functionality. | We will require to introduce new code and a new structure in order to achieve the functionality. |
| Migration requirements | Yes | None |

### Decision 2: How should Android platform parameters be modeled in storage?

**Problem Statement:**  
Android platform parameters must:

* Be evaluated using Android-specific app version and flavour.  
* Be managed separately from Web parameters.  
* Support Android \-specific override conditions.  
* Be editable only by Android Release Coordinators.

We must decide whether Android and Web platform parameters should share a single model or use separate models.

We have considered the following alternatives:

1. **Introduce AndroidPlatformParameterConfigModel**  
2. **Reuse Web PlatformParameterModel (Web Model)**

Among these, we believe that first is the best approach, because:

* Web and Android parameters have different evaluation contexts.  
  * Web uses platform\_type \+ app\_version \+ app\_version\_flavour+server\_mode  
  * Android uses app\_min\_ version+app\_max\_version \+ app\_version\_flavour  
* Access control requirements differ:  
  * Web parameters are managed by SuperAdmin  
  * Android parameters are managed by Android RCs.  
* Separate models simplify registry and handler logic  
* In the future if we plan to have some properties that need to be associated with android platform parameters  or web platform parameters, we can easily update the relevant models to achieve it

The above approaches are contrasted in detail in the following table:

|  | Reuse Existing Model (Web PlatformParameterModel) | New Model |
| :---- | :---- | :---- |
| Code & system maintainability and readability | Making changes to the existing **PlatformParameterModel** will mix-up the properties of both android platform parameters and web platform parameters. Properties for web platform parameters are: rules rule\_schema\_version default\_value Properties for android feature-flags: rules rule\_schema\_version default\_value Even though the storage fields of web and Android platform parameters are currently identical, reusing the same model is not the right design choice because: Field Web Platform Parameter Android Platform Parameter rules Apply based on: \- platform\_type \- server\_mode \- app\_version Apply based on: \- app\_min\_version \- app\_max\_version \- app\_version\_flavour rule\_schema\_version Evolves with web config rules. Evolves with Android config rules. default\_value Web fallback config. Android fallback config. In future if there are some properties that are unique to web platform parameters and android platform parameters, we would require to add extra comments, validations to make sure that the properties are correctly assigned. | With the help of a new model we will be able to separate the code for android platform parameters and web platform parameters. It will be a lot easier to maintain the functionalities of android platform parameters and web-platform parameters. The readability will automatically increase. In future if there are some properties that are unique to android platform parameters and web platform parameters, it will be easier to implement. |
| Consistency with existing codebase | We will be able to use the existing code and structure to implement the functionality. | We will require to introduce new code and a new structure in order to achieve the functionality. |
| Registry, Service and Handler clarity | On reusing: One service must serve two clients. Harder to test and debug for failures. Adding a new entry needs multiple changes in the codebase. | With separate models: \- Have simpler interfaces. \- Avoid branching logic. |
| Migration requirements | Yes | No (Only need to remove Android references from [platform\_parameter\_domain.py](https://github.com/oppia/oppia/blob/fb73561a6f321670be8191bb194c64d130417cb3/core/domain/platform_parameter_domain.py#L17)) |

### 

### Decision 3: How should Android Feature Flags and Platform Parameters be registered?

**Problem Statement:**  
We need a reliable mechanism to:

* Define all valid Android feature flags and platform parameters.  
* Prevent typos and unregistered flags from being served or modified.  
* Provide a single source of truth for Android-only configuration.  
   

We must decide whether Android configuration should reuse existing web registries or introduce Android-specific registries.

We have considered the following alternatives:

1. **Reuse existing Web registries ([platform\_parameter\_list.py](https://github.com/oppia/oppia/blob/fb73561a6f321670be8191bb194c64d130417cb3/core/domain/platform_parameter_list.py#L17), [feature\_flag\_list.py](https://github.com/oppia/oppia/blob/fb73561a6f321670be8191bb194c64d130417cb3/core/feature_flag_list.py#L17))**  
2. **Introduce separate Android registries.**  
   1. **android\_feature\_flag\_registry.py**  
   2. **android\_platform\_parameter\_registry.py**

Among these, we believe that second is the best approach, because:

* Different lifecycle rules:  
  * Web flags are dropped after rollout.  
  * Android flags persist as LIVE → FINAL and remain forever.  
  * A shared registry would force incompatible lifecycle semantics.  
* Different Consumers:  
  * Web registry serves Web backend.  
  * Android registry serves android clients.  
* Clear validation boundary:  
  * Prevents Android handlers from accidentally reading Web config.  
  * Prevents Web handlers from reading Android config.  
* Easier to maintain and understand the whole registry process (will follow the same pattern followed by Web registries).

The above approaches are contrasted in detail in the following table:

|  | Reuse existing Web registries | Introduce separate Android registries. |
| :---- | :---- | :---- |
| Code & system maintainability and readability | \- Mixing Android and Web configuration in the same registries makes the codebase harder to understand. \- Developers must keep track of which entries apply to which platform, increasing load and risk of bugs. \- Ownership of that file is complex, as it contains config for both Web and Android. | \- Having android only registries provides a clear interface for adding or modifying Android configurations.   \- This improves readability and is easier to maintain and understand. \- Ownership of separated files is easier and will be maintained by the android team. |
| Consistency with existing codebase | We already have separate registries for platform parameters and feature flags for the web. Merging respective android config will cause a similar issue of having entities for different platforms in one file.And in future we might need to refactor it.  | We will require to introduce new code and a new structure in order to achieve the functionality. |
| Risk of Cross-Platform Bugs | HIGH, Accidental use of web config for Android or vice-versa | LOW, Android config is separated, making misuse difficult. |

We will introduce separate Android registries:

* android\_feature\_flag\_registry.py  
* android\_platform\_parameter\_registry.py

This ensures clean separation of concerns, safer evolution, clearer ownership, and significantly improved maintainability.

## Risks and mitigations

| Potential Risk | Mitigation |
| :---- | :---- |
| Unauthorized users gaining access to Android release coordinator actions. | Introduced a new role: Android release Coordinator.All Android admin handlers will be protected using a new ACL decorator can\_access\_android\_release\_coordinator\_page |
| Malformed app-version or installation ID headers | \- All incoming headers are validated server-side. \- Installation IDs and app-version are never stored. |
| Rollout manipulation by clients | \- Rollout evaluation is fully server side, deterministic and salted with feature flag name.\- Clients cannot influence rollout eligibility beyond their installation ID. |
| Feature rollout causing widespread feature enablement/disablement | \- Rollout computation is deterministic, stateless, and covered by unit tests. \- Salting with the feature flag name ensures independence across features. |
| Future contributors accidentally modifying the wrong configuration. | \- Dashboards, handlers and models, registries and all related files are explicitly named with prefixes of web and android respectively. \- Access control ensures only Android RCs can modify Android configuration. |
| Confusion between **“flag absent” vs "flag off” semantics** | \- Handlers explicitly differentiate these cases. \- Absence means “client decides”; \- presence with **enabled= false** , means “server explicitly disables”. |
| FINAL feature flags being modified | \- Frontend keeps that section disabled from any input. \- Domain layer validation strictly prevents updates to the FINAL feature flag. \- Any attempt to modify it raises a validation error. |

## Implementation Approach

### **\[Web only\]** Storage Model Layer Changes

1. **BaseFeatureFlagConfigModel** (New \- Abstract)  
   * **Location** : core/storage/config/gae\_models.py  
   * **Purpose:** Defines shared behaviour for feature flag configuration models across web and Android.  
   * **Source of truth**: Yes.  
   * **ID generation**:  
     * id:str  
     * The ID is the feature flag name, which is globally unique and defined in the corresponding registry.  
   * **Fields**:  
     * **id**: str  
       * Feature flag name.  
     * **created\_on**: datetime  
       * Inherited from [BaseMode](https://github.com/oppia/oppia/blob/1c04360467ab3935d199f02010e3bfce9a4ce819/core/storage/base_model/gae_models.py#L160)l. Timestamp of initial creation.  
     * **last\_updated**: datetime  
       * Inherited from BaseModel. Timestamp of last update.  
     * **deleted**: bool  
       * Inherited from BaseModel. Indicates deletion type (soft/hard).  
   * **Validation constraints:**  
     * ID must match a registered feature flag name.  
     * Platform-specific validation(web / Android) is enforced in child models.  
   * **Takeout / Wipeout**:  
     * Not associated with any user data.  
     * No Takeout or Wipeout support required.  
   * **NOTES:**  
     * This model is never instantiated directly.  
     * All feature-flag storage models must extend this class.

| class BaseFeatureFlagConfigModel(base\_models.BaseModel):     """Abstract base model for feature flag configuration.     This model defines the common structure for all feature flags across     Web and Android. It must not be instantiated directly.     The id field represents the globally unique feature flag name.     Fields:         id: str. Unique name of the feature flag.         created\_on: datetime. Time of creation.         last\_updated: datetime. Time of last update.         deleted: bool. Whether the model is marked deleted.     """  |
| :---- |

2. **AndroidFeatureFlagConfigModel** (New Model)  
* Location: core/storage/config/gae\_models.py  
* Extends **BaseFeatureFlagConfigModel.**  
* Purpose: Stores Android-only feature flag configuration.  
* ID Generation:  
  * id: str (Feature flag name)  
  * Inherited from **BaseFeatureFlagConfigModel.**  
* **Fields:**  
  * **Id: str**  
    * Android Feature flag name.  
  * **state : str**  
    * **Enum: LIVE | FINAL**  
    * **LIVE** : The feature flag is configurable and can be updated.  
    * **FINAL**: The feature flag is locked and immutable.  
  * **min\_app\_version: str**  
    * Minimum Android app version for which the flag applies.  
  * **max\_app\_version: Optional\[str\]**  
    * Maximum Android app version for which the flag applies.  
  * **rollout\_percentage: int**  
    * Percentage rollout \[0-100\]  
  * **created\_on**: datetime  
    * Inherited from [BaseMode](https://github.com/oppia/oppia/blob/1c04360467ab3935d199f02010e3bfce9a4ce819/core/storage/base_model/gae_models.py#L160)l. Timestamp of initial creation.  
  * **last\_updated**: datetime  
    * Inherited from BaseModel. Timestamp of last update.  
  * **deleted**: bool  
    * Inherited from BaseModel. Indicates deletion type (soft/hard)

.

* **Validation constraints:**  
  * ID must match a registered Android feature flag name.  
  * state must be one of {LIVE , FINAL}.  
  * rollout\_percentage must be an integer in \[0,100\].  
  * max\_app\_version (If defined) must be \>= min\_app\_version.  
  * Flags in FINAL state must not be updated.  
  * Web-only fields must be None.  
  * Take reference from here([core/domain/feature\_flag\_domain.py](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L56))  
  * For more info [see this Bookmark](#bookmark=id.1c6rb9pq6jsd).

* **Takeout / Wipeout**:  
  * Not associated with any user data.  
  * No Takeout or Wipeout support required.

* **Datastore operations:** All datastore access is **centralized in the registry/service** layer; caller must not access the model directly outside these layers.  
1.  **get(id, strict=False):** Fetch a single Android feature flag configuration by its name.

   \- Returns the corresponding **AndroidFeatureFlagConfigModel** if present.

   \- Returns **None** if no configuration exists for the given feature flag name.

   \- Does not raise if the entity is missing.

    

2. **get\_multi(ids):** Fetch Android feature flag configurations for a list of feature flag names.

			  
			\- Accepts a list of feature flag names (IDs).  
			\- Returns a list of models aligned with the input order.  
			\- Missing entities are returned as **None.**  
		  
			**Note: Input size must not exceed datastore limits.**

3. **put():** Update mutable fields of an existing Android feature flag configuration.

   \- Updates are allowed only when **state \== LIVE**.

   \- Flags in FINAL state are immutable.

   \- All validation rules are re-checked before writing.

		**Error handling:**  
\- Validation of feature flag existence is handled at the registry/domain layer.  
\- Storage layer does not raise errors for missing entities.

| class AndroidFeatureFlagConfigModel(BaseFeatureFlagConfigModel):   """Storage model for Android-specific feature flag configuration.     This model stores configuration data used to evaluate feature flags     for the Android application only. It extends BaseFeatureFlagConfigModel     and must only be used for Android feature flags.     The id field represents the unique feature flag name and must correspond     to a registered Android feature flag in the Android feature flag registry.     Fields:         id: str. Unique name of the Android feature flag.         state: str. The lifecycle state of the feature flag. Allowed values             are:             \- LIVE: The feature flag is configurable and may be updated.             \- FINAL: The feature flag is locked and must not be modified.         min\_app\_version: str. The minimum Android app version for which the             feature flag applies.         max\_app\_version: Optional\[str\]. The maximum Android app version for             which the feature flag applies. If None, the flag applies to all             versions greater than or equal to min\_app\_version.         rollout\_percentage: int. Percentage of users for which the feature             flag is enabled. Must be in the range \[0, 100\].         created\_on: datetime. Time of initial creation.         last\_updated: datetime. Time of last update.         deleted: bool. Whether the model is marked deleted.     """ |
| :---- |

3\. **WebFeatureFlagConfigModel** (Renamed Model)

* Location:  core/storage/config/gae\_models.py  
* Extends: **BaseFeatureFlagConfigModel.**  
* Purpose: Stores Web-only feature flag configuration.  
* ID Generation:  
  * id: str (Feature flag name)  
  * Inherited from **BaseFeatureFlagConfigModel.**  
* **Fields:**  
  * **Id: str**  
    * Web Feature flag name.  
  * **force\_enable\_for\_all\_users: bool**  
    * Whether the feature flag is force-enabled for all users, bypassing rollout and user-group-based evaluation.  
  * **rollout\_percentage: int**  
    * Percentage rollout \[0-100\]  
  * **user\_group\_ids: List\[str\]**  
    * List of **UserGroupModel IDs** for which the feature flag is enabled.  
  * **created\_on**: datetime  
    * Inherited from [BaseMode](https://github.com/oppia/oppia/blob/1c04360467ab3935d199f02010e3bfce9a4ce819/core/storage/base_model/gae_models.py#L160)l. Timestamp of initial creation.  
  * **last\_updated**: datetime  
    * Inherited from BaseModel. Timestamp of last update.  
  * **deleted**: bool  
    * Inherited from BaseModel. Indicates deletion type (soft/hard)  
* **Validation constraints:** Already implemented [here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L56)  (will rename this file with web as prefix)  
* **Takeout / Wipeout**:  
  * Not associated with any user data.  
  * No Takeout or Wipeout support required.  
* **Datastore Operations:**  
  * **get(id, strict=False)**  
  * **get\_multi(ids)**  
  * **put()**

Notes:

- This model is renamed from the existing **FeatureFlagConfigModel**.  
- All related services, domain files, and functions will be renamed to explicitly indicate web-only usage (prefix: web).  
- No datastore migration is required, as the underlying schema and IDs remain unchanged.

| class WebFeatureFlagConfigModel(BaseFeatureFlagConfigModel):      """ Storage model for Web-specific feature flag configuration.     This model stores configuration data used to evaluate feature flags     for the web platform only. It extends BaseFeatureFlagConfigModel and     must only be used for Web feature flags.     The id field represents the unique feature flag name and must correspond     to a registered Web feature flag in the web feature flag registry.     Fields:         id: str. Unique name of the Web feature flag.         force\_enable\_for\_all\_users: bool. Whether the feature flag is             force-enabled for all users.         rollout\_percentage: int. Percentage of logged-in users for which             the feature flag is enabled. Must be in the range \[0, 100\].         user\_group\_ids: List\[str\]. List of user group IDs for which the             feature flag is enabled.         created\_on: datetime. Time of initial creation.         last\_updated: datetime. Time of last update.         deleted: bool. Whether the model is marked deleted.     """ |
| :---- |

**4\. BasePlatformParameterConfigModel** (New \- Abstract)

* Location: core/storage/config/gae\_models.py  
* Purpose: Defines shared storage structure for platform parameter configuration across Web and Android.  
* Source of truth: Yes  
* **ID generation**:  
  * id:str  
  * The ID is the platform parameter name, which is globally unique and defined in the corresponding platform parameter registry.  
* **Fields**:  
  * **id**: str  
    * Platform parameter name.  
  * **created\_on: datetime**  
    * Inherited from VersionedModel. Timestamp of initial creation.  
  * **last\_updated: datetime**  
    * Inherited from VersionedModel. Timestamp of last update.  
  * **deleted**: bool  
    * Inherited from VersionedModel. Indicates deletion type (soft/hard).  
* **Validation constraints:**  
  * ID must match a registered platform parameter name.  
  * Platform-specific validation(web / Android) is enforced in child models.  
* **Takeout / Wipeout**:  
  * Not associated with any user data.  
  * No Takeout or Wipeout support required.  
* **NOTES:**  
  * **Versioning & Snapshots:**  
    * **BasePlatformParameterConfigModel** extends **VersionedModel.**  
    * Therefore, **both Web and Android platform parameters are versioned.**  
    * Existing snapshot models are reused:  
      * [**PlatformParameterSnapshotMetadataModel**](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/storage/config/gae_models.py#L37)  
      * [**PlatformParameterSnapshotContentModel**](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/storage/config/gae_models.py#L45C6-L45C44)  
  * This model is never instantiated directly.  
  * All platform parameter storage models must extend this class.

| class BasePlatformParameterConfigModel(base\_models.VersionedModel):     """Abstract base model for platform parameter configuration.     This model defines the common storage structure for platform parameters     across Web and Android. It must not be instantiated directly.     The id field represents the unique platform parameter name.     Fields:         id: str. Unique name of the platform parameter.         created\_on: datetime. Time of creation.         last\_updated: datetime. Time of last update.         deleted: bool. Whether the model is marked deleted.     """  |
| :---- |

**5\. AndroidPlatformParameterConfigModel** (New Model)

* Location: core/storage/config/gae\_models.py  
* Purpose: Stores Android-only platform parameter configuration.  
* Source of truth: Yes  
* Extends: **BasePlatformParameterConfigModel**  
* **Fields**:  
  * **rules**: List\[dict\]  
    * List of dict representations of **AndroidPlatformParameterRule** objects.  
    * Each rule has the following structure:  
      * value\_when\_matched: The value returned when the rule matches.  
      * filters: List\[dict\]  
        * List of **AndroidPlatformParameterFilter** representations, each containing:  
          * type: str —filter type  
          * conditions: List\[(str, str)\] — (operator, value) pairs  
  * rule\_schema\_version: int  
    * Schema version for interpreting Android rule dictionaries.  
  * default\_value: Any  
    * default value of the platform parameter.

* **Validation constraints:**  
  * ID must match a registered platform parameter name.  
  * Platform-specific validation Android is enforced in child models  
  * For more info [see this bookmark](#bookmark=id.fk8zih85184f)..  
* **Takeout / Wipeout**:  
  * Not associated with any user data.  
  * No Takeout or Wipeout support required.  
* **Datastore operations:** All datastore access is centralized in the registry and service layers; callers must not access the model directly outside these layers.  
  * **Datastore operations:** All datastore access is **centralized in the registry/service** layer; caller must not access the model directly outside these layers.

1.  **get(id, strict=False):** Fetch a single Android Platform parameter configuration by its name.

   \- Returns the corresponding **AndroidPlatformParameter** if present.

   \- Returns **None** if no configuration exists for the given platform parameter name.

   \- Does not raise if the entity is missing.

    

2. **commit(commiter\_id, commit\_message, commit\_cmds):** Persists updates to an existing platform parameter configuration.  
* Creates a new version of Platform parameter.  
* Automatically creates a snapshot metadata and snapshot content entries.  
* Updates last\_updated and version metadata.  
* All validation rules are re-checked at the domain layer before committing.

			**Note:** 

- Platform parameters extend VersionedModel; therefore, direct put() calls are not used.  
- All writes must go through commit() to preserve versioning and snapshot history.

3. **create(id, rules, rule\_schema\_version, default\_value):** Creates a new platform parameter config, if it does not already exist.

   \- Initializes the storage model with the given rules and default value.

   \- Internally performs an initial write to the datastore.

   \- Used only during first-time creation or bootstrap.

* **NOTES:**  
  * Validation of Platform parameter existence and correctness is handled at the registry and domain layers.  
  * Invalid rule structures, schema mismatches, or data-type violations raise validation errors before persistence.  
  * All related services, domain files, and functions will be named to explicitly indicate Android-only usage (prefix: android).

| class AndroidPlatformParameterConfigModel(BasePlatformParameterConfigModel):     """Storage model for Android-specific platform parameter configuration.     This model stores configuration data used to evaluate platform parameters     for the Android application only.     The id field represents the unique platform parameter name and must     correspond to a registered Android platform parameter.     Fields:         rules: list(dict). List of dict representation of                 AndroidPlatformParameterRule objects, which have the following                 structure:                     \- value\_when\_matched:  The result of the rule when it's matched.                     \- filters: list(dict). List of dict representation of AndroidPlatformParameterFilter               objects, having the following structure:                             \- type: str. The type of the filter.                             \- conditions: list((str, str)). Each element of the                                 list is a 2-tuple (op, value), where op is the                                 operator for comparison and value is the value                                 used for comparison. rule\_schema\_version: The schema version for the rule dicts. default\_value: The default value of the platform parameter         Returns:             AndroidPlatformParameterModel. The created AndroidPlatformParameterModel             instance.     """  |
| :---- |

**6\. WebPlatformParameterConfigModel** (Renamed Model)

* Location: core/storage/config/gae\_models.py  
* Purpose: Stores Web-only platform parameter configuration.  
* Source of truth: Yes  
* Extends: **BasePlatformParameterConfigModel**  
* **Fields**:  
  * **rules**: List\[dict\]  
    * List of dict representations of [**WebPlatformParameterRule**](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L570) objects.  
    * Each rule has the following structure:  
      * value\_when\_matched: The value returned when the rule matches.  
      * filters: List\[dict\]  
        * List of **[WebPlatformParameterFilter](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L255)** representations, each containing:  
          * type: str —filter type  
          * conditions: List\[(str, str)\] — (operator, value) pairs  
  * rule\_schema\_version: int  
    * Schema version for interpreting web rule dictionaries.  
  * default\_value: Any  
    * default value of the platform parameter.

* **Validation constraints:**  
  * Already implemented [here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L17)  (will rename this file with android as prefix)  
* **Takeout / Wipeout**:  
  * Not associated with any user data.  
  * No Takeout or Wipeout support required.  
* **Datastore operations:**  
  * **get(id, strict=False)**  
  * **create()**  
  * **commit()**  
* **NOTES:**  
  * All related services, domain files, and functions will be named to explicitly indicate Android-only usage (prefix: android).

| class WebPlatformParameterConfigModel(BasePlatformParameterConfigModel):       """Creates a WebPlatformParameterModel instance.         Args:             param\_name: str. The name of the web-parameter, which is immutable.             rule\_dicts: list(dict). List of dict representation of                 WebPlatformParameterRule objects, which have the following                 structure:                     \- value\_when\_matched: \*. The result of the rule when it's                         matched.                     \- filters: list(dict). List of dict representation of                         WebPlatformParameterFilter objects, having the following                         structure:                             \- type: str. The type of the filter.                             \- conditions: list((str, str)). Each element of the                                 list is a 2-tuple (op, value), where op is the                                 operator for comparison and value is the value                                 used for comparison.             rule\_schema\_version: int. The schema version for the rule dicts.             default\_value: WebPlatformDataTypes. The default value of the platform                 parameter.         Returns:             WebPlatformParameterModel. The created WebPlatformParameterModel             instance.         """  |
| :---- |

### **\[Web only\]** Storage Model Migrations

**Migration Requirements**: No datastore migration is required.  
**Beam jobs**: As there is no need to backfill, No beam jobs are required.

**Justification:**

- **WebPlatformParameterConfigModel** is a rename of the existing **PlatformParameterModel**.  
- **WebFeatureFlagConfigModel** is a rename of the existing **FeatureFlagConfigModel**.  
- The underlying datastore schema remains unchanged:  
  - Same entity kind  
  - Same ID format (**Platform Parameter name, Feature flag name**)  
  - Same fields (**rules, rule\_schema\_version, default\_value, force\_enable\_for\_all\_users,rollout\_percentage,user\_group\_ids**)  
- Only code-level renaming and refactor is performed to make the Web-only nature explicit.  
- The refactor only modifies the inheritance hierarchy and naming for clarity, which does not affect datastore schema or existing production data.

### Domain Objects

To avoid cross-platform coupling, Web and Android use separate domain objects for both feature flags and platform parameters.

* **Location:** `core/domain/feature_flag_domain.py` (renamed logically to `web_feature_flag_domain.py`)  
* WebFeatureFlag (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L71))  
* WebFeatureFlagDict (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L59))  
* WebFeatureFlagSpec (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L180))  
* WebFeatureFlagSpecDict (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L173))  
* WebFeatureFlagConfig (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L254))  
* WebFeatureFlagConfigDict (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L245))

* **Location:** `core/domain/platform_parameter_domain.py` (renamed logically to `web_platform_parameter_domain.py`)  
* WebPlatformParameter (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L667))  
* WebEvaluationContext (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L130))  
* WebPlatformParameterRule (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L570))  
* WebPlatformParameterFilter (Renamed, [existing here](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/platform_parameter_domain.py#L255))

* Location: `core/domain/android_feature_flag_domain.py (New)`  
* **AndroidFeatureFlagState (enum.Enum):**


| “”” Enum representing the lifecycle state of an Android feature flag ””” |
| :---- |

- **LIVE**: The feature flag is configurable and can be updated.  
- **FINAL:** The feature flag is locked and must not be modified.

* **AndroidFeatureFlagDict (TypedDict):**


| “”” Dictionary representing AndroidFeatureFlag object. ””” |
| :---: |

- name: str  
- state: str  
- min\_app\_version: str  
- max\_app\_version: Optional\[str\]  
- rollout\_percentage: int

* **AndroidFeatureFlag:**

| """AndroidFeatureFlag domain object.""" |
| :---: |

- Fields:  
  - **name: str**

				Unique name of the Android Feature flag.

- **state: AndroidFeatureFlagState**

				ENUM: LIVE | FINAL

- **min\_app\_version: str**

  The minimum Android app version for which the feature flag applies.

- **max\_app\_version: optional\[str\]**

  The maximum Android app version for which the feature flag applies. If None, the flag applies to all versions greater than or equal to min\_app\_version.

- **rollout\_percentage: int**

  Percentage of users for which the feature flag is enabled. Must be in the range \[0, 100\]


- Methods:  
1. **validate(self)**

| """Validates the AndroidFeatureFlag object."""" |
| :---: |

			Validation rules:

- ID must match a registered Android feature flag name.  
- state must be one of {LIVE , FINAL}.  
- rollout\_percentage must be an integer in \[0,100\].  
- min\_app\_version must be defined and \>= min\_app\_version.  
- max\_app\_version (If defined) must be \>= min\_app\_version.  
- Flags in FINAL state must not be updated.  
- Web-only fields must be None.  
- Take reference from here([core/domain/feature\_flag\_domain.py](https://github.com/oppia/oppia/blob/36ad2b2a3ff561c61ed9b56968b81ce8f4e37b71/core/domain/feature_flag_domain.py#L56))  
- For more info [see this Bookmark](#bookmark=id.1c6rb9pq6jsd).

			  
			Raises

- **ValidationError** if any validation rule is violated.

2. **from\_dict(self, android\_feature\_flag\_dict: AndroidFeatureFlagDict)**

   

|        """Returns an AndroidFeatureFlag object from dictionary.         Args:             android\_feature\_dict: dict. A dict mapping of all fields of                 AndroidFeatureFlag object.         Returns:             AndroidFeatureFlag. The corresponding FeatureFlag domain object.         """ |
| :---- |

   

3. **to\_dict(self)**

| """Returns a dict representation of the AndroidFeatureFlag domain object.         Returns:             dict. A dict mapping of all fields of AndroidFeatureFlag object.         """ |
| :---- |

4. **serialize(self)**

|        """Returns the object serialized as a JSON string.         Returns:             str. JSON-encoded string encoding all of the information composing             the object.         """ |
| :---- |

5. **deserialize(self, json\_string:str)**

   

|        """Returns a AndroidFeatureFlag domain object decoded from a JSON         string.         Args:             json\_string: str. A JSON-encoded string that can be                 decoded into a dictionary representing a AndroidFeatureFlag.                 Only call on strings that were created using serialize().         Returns:             AndroidFeatureFlag. The corresponding AndroidFeatureFlag domain             object.         """ |
| :---- |

   

	**Note:** 

- **Serialize() method** gets used in the **caching\_services.py** file as we need serialization of python objects for the caching.  
- **Deserialize()**  method gets used in the **caching\_services.py** file as we need deserialization to return a python object from a JSON string.  
- This will mirror the design of PlatformParameter domain objects.

* Location: `core/domain/android_platform_parameter_domain.py (New)`  
     
* **AndroidPlatformDataTypes** \= Union\[str, int, bool, float\]  
* **AndroidPlatformParameterFilterDict (TypedDict):**


| """Dictionary representing AndroidPlatformParameterFilter object.""" |
| :---: |

- **type: str**  
- **conditions: List\[List\[str\]\]**

* **AndroidEvaluationContext:**

| “””Context used for evaluating Android platform parameters.””” |
| :---: |


- **Fields:**  
  - **app\_version: str**  
  - **app\_version\_flavour: str**  
- **Methods:**  
  - **validate(self) \-\> None**

|  """ Validates the AndroidEvaluationContext domain object, raising an exception if the object is in an irrecoverable error state. """ |
| :---- |

    

* **AndroidPlatformParameterRuleDict(TypedDict):**


| """Dictionary representing AndroidPlatformParameterRule object.""" |
| :---: |


- **filters:** List\[AndroidPlatformParameterFilterDict\]  
- **value\_when\_matched:** AndroidPlatformDataTypes

* **AndroidPlatformParameterRule:**


| """Domain object representing a rule for Android platform parameters.""" |
| :---: |


- **filters:** List\[AndroidPlatformParameterFilter\]  
- **value\_when\_matched:** AndroidPlatformDataTypes

		Methods:

- **validate(self, data\_type: str) \-\> None**  
  - Ensures value\_when\_matched matches the parameter’s data type.  
  - Validates all filters.  
- **evaluate(self, context: AndroidEvaluationContext) \-\> bool**  
  - A rule matches **only if all filters match.**

    

* **AndroidPlatformParameterFilter:**

| """Domain object representing a filter for Android platform parameters.""" |
| :---: |

    
   **\-   SUPPORTED\_FILTER\_TYPES: Final \= \[**

          **'app\_min\_version',**

          **'app\_max\_version’',**

          **'app\_version\_flavour',**

      **\]**  
  **\-    SUPPORTED\_OP\_FOR\_FILTERS: Final \= {**

          **'app\_min\_version': \['='\],**

          **'app\_max\_version': \['='\],**

          **'app\_version\_flavour': \['='\],**

      **}**  
     
  **Methods:**

1. **validate(self) \-\> None**

| “””Validates the filter definition.””” |
| :---: |

   **Validation rules:**

- Filter type must be supported.  
- Operators must be valid for the given filter type.  
- Version expressions must be valid numeric version strings.  
- Flavour must be one of the allowed Android flavours.

			**Raises:**

- Raises **ValidationError** if invalid.

2. **evaluate(self, context: AndroidEvaluationContext) \-\> bool**

|        """Tries to match the given context with the filter against its         value(s).         Args:             context: AndroidEvaluationContext. The context for evaluation.         Returns:             bool. True if the filter is matched.         """ |
| :---- |

   

* **AndroidPlatformParameterDict (TypedDict):**


| “”” Dictionary representing AndroidPlatformParameter object. ””” |
| :---: |

- name: str  
- data\_type: str  
- rules: List\[AndroidPlatformParameterRuleDict\]  
- rule\_schema\_version: int  
- default\_value: AndroidPlatformDataTypes

* **AndroidPlatformParameter:**


| “”” AndroidPlatformParameter domain object. ””” |
| :---: |

- **name: str**  
  Unique name of the Android platform parameter.  
- **data\_type: str**  
  Data type of the parameter. Must be one of bool, string, float, int  
- **rules: List\[PlatformParameterRuleDict\]**  
  Ordered list of Android platform parameter rules.  
- **rule\_schema\_version: int**  
  Schema version for interpreting rule dictionaries.  
- **default\_value: PlatformDataTypes**

			Default value returned when no rule matches.  
		Methods:

1. **validate(self) \-\> None**

| “”” Validates the AndroidPlatformParameter object.””” |
| :---- |

   

   **Validation rules:**

- Name must match platform parameter naming regexp.  
- default\_value must be present  
- data\_type must be supported.  
- default\_value must match data\_type.  
- Each rule:  
  - Must define value\_when\_matched.  
  - Must match data\_type.  
    **Raises:**  
    		**ValidationError** if invalid.

      
- **evaluate(self, context: AndroidEvaluationContext) \-\> AndroidPlatformDataTypes:**


| “”” Evaluates the Android platform parameter for the given context ””” |
| :---- |

    
  **Evaluation logic:**

- Rules are evaluated **top-down.**  
- The **first matching rule** returns its value.  
- If no rule matches, **default\_value** is returned.


2. **to\_dict(self) \-\> AndroidPlatformParameterDict**  
   	

| """Returns a dict representation of the AndroidPlatformParameter domain object.         Returns:             dict. A dict mapping of all fields of AndroidPlatformParameter object.         """ |
| :---- |

   

3. **from\_dict(self, param\_dict: AndroidPlatformParameterDict ) \-\> AndroidPlatformParameter**

   

|  """Returns an AndroidPlatformParameter object from dictionary.         Args:             android\_feature\_dict: dict. A dict mapping of all fields of                 AndroidPlatformParameter object.         Returns:             AndroidPlatformParameter. The corresponding Platform parameter domain object. “”” |
| :---- |

4. **seralize(self) \-\> str**

   

|        """Returns the object serialized as a JSON string.         Returns:             str. JSON-encoded string encoding all of the information composing             the object.         """ |
| :---- |

   

   

5. **deserialize(self, json\_string:str)**

   

|        """Returns a AndroidPlatformParameter domain object decoded from a JSON         string.         Args:             json\_string: str. A JSON-encoded string that can be                 decoded into a dictionary representing a AndroidPlatformParameter.                 Only call on strings that were created using serialize().         Returns:             AndroidPlatformParameter. The corresponding AndroidPlatformParameter domain             object.         """ |
| :---- |

   

	**Note:** 

- **Serialize() method** gets used in the **caching\_services.py** file as we need serialization of python objects for the caching.  
- **Deserialize()**  method gets used in the **caching\_services.py** file as we need deserialization to return a python object from a JSON string.


### User Flows (Controllers and Services)

### Web feature flags and platform parameters are only renamed and do not introduce new behaviour.  Android functionality is split into two verticals:

1. Runtime evaluation for Android clients (read-only).  
2. Authoring and updates via Android Release Coordinator (admin-only)

All operations are synchronous. No background tasks or taskqueues are introduced.

Android Feature Flags:

- **Files Introduced:**  
  - **core/domain/android\_feature\_flag\_domain.py**  
    - Domain objects: AndroidFeatureFlag, AndroidFeatureFlagState, AndroidFeatureFlagDict  
  - **core/domain/android\_feature\_flag\_registry.py**  
    - Registry for Android feature flags.  
    - Methods: get\_android\_feature\_flag(), update\_feature\_flag(), load\_feature\_flag\_from\_storage()  
  - **core/domain/android\_feature\_flag\_services.py**  
    - Service layer: get\_evaluated\_android\_feature\_flags(), update\_android\_feature\_flag(), get\_all\_android\_feature\_flags().  
- **core/controllers/android\_feature\_flag\_handler.py**  
  - Handlers: AndroidFeatureFlagsAdminHandler

- **User Flow 1: App requests Feature Flag Evaluation:**

  **Scenario:** Android app requests all applicable feature flags for its current version and installation ID.

  **Endpoint:** GET /android\_feature\_flags

**Access control:** open\_access  
**Headers:**

- app\_version\_name  
- installation\_id  
- app\_package\_name

- **Controller Flow:**  
  - **Handler: AndroidFeatureFlagsHandler**

| def get(self) \-\> None:“”” Handles GET requests for evaluated Android Feature Flags””” |
| :---- |

    

- **Pseudocode:**  
1. **Extract request headers:**  
> > > > > > - app\_version\_name  
> > > > > > - installation\_id  
> > > > > > - app\_package\_name  
2. **Validate that all required headers are present**  
> > > > > > - If missing : **InvalidInputException**  
3. **Call service layer:**  
> > > > > > - **evaluated\_flags \= android\_feature\_flag\_services.get\_evaluated\_android\_feature\_flags( app\_version\_name, installation\_id)**  
4. **Return JSON response:**

   **{**

   	**“feature\_flags”: evaluated\_flags**

   **}**

   **\- Execution type:** Synchronous.

- **Service Layer:**  
  **Function: android\_feature\_flag\_services.get\_evaluated\_android\_feature\_flags()**

| def get\_evaluated\_android\_feature\_flags(          app\_version\_name: str,          Installation\_id: str ) \-\> List\[Dict\[str, bool\]\]: “”” Returns evaluated Android feature flags for a given app\_version and installation\_id.Evaluates all registered Android feature flags and returns only those applicable to the specified app version. Flags are filtered based on version constraints and app flavour. Rollout percentage is applied for LIVE flags in GA flavour.Args:      App\_version\_name: str. Android app version string.       Installation\_id: str. Returns:        list(dict). List of dictionaries of the form:             {                   “name”: str Feature flag name.                   “enabled”: bool. Whether feature is enabled.             }Note: Omitted flags (version mismatch) are NOT included. This differs from returning {“enabled”: false} which explicitly disables.Raises:     Exception. If app\_version\_name format is invalid.“”” |
| :---- |


- **Evaluation Logic:**  
  	1\. Fetch all registered Android feature flag names from the registry.  
  		\- **feature\_flag\_names \=android\_feature\_flag\_registry.get\_all\_feature\_flag\_names()**  
  2\. Load feature flag configs from storage:  
  		\- **feature\_flag\_dict \= load\_android\_feature\_flags\_from\_storage (feature\_flag\_names)**  
  		**\- \[DATASTORE: GET\_MULTI 1 Call\]**  
  3\. Parse app\_version\_name into:  
  		\- version number

			\- flavour  
		4.Initialize empty response list.  
			evaluated\_flags \= \[\]

		5\. For each feature flag config:

1. If app\_version \< feature\_flag.min\_app\_version:

   \-\> continue (omit flag )

2. If max\_app\_version is defined and app\_version \> feature\_flag.max\_app\_version

  			\-\> continue (omit flag )

3. If feature\_flag.state \== LIVE and flavour \!= GA:  
   \-\> continue (omit flag)

   At this point, the flag is eligible to be returned.

4. If feature\_flag.state \== FINAL:  
   \-\> enabled \= True

   \-\> add {name, enabled} to response.

   \-\> continue

   

5. (flag.state \== LIVE AND flavour \== GA):

   \-\> compute rollout bucket using installation\_id

   \-\> enabled \= bucket \< rollout\_percentage

   \-\> add {name, enabled} to response

   

		6\. Return response list.

**NOTE: Returning a flag as enabled= false is not equivalent to omission.**

| Case | Meaning |
| :---- | :---- |
| Flag omitted | Web Server has no opinion → app decides |
| Flag returned with enabled \= false. | Server explicitly disables. |

Version mismatch always results in omission, never **“enabled”: false.**

- Also refer this table for flavour based filtering [TDD - Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1OyT2jfL-2K3gZ3weWLQhRtjxqfwkfcADM3I1xAObFas/edit?tab=t.0#bookmark=id.mlod0tcrdwdo)

**Datastore Calls:** GET\_MULTI count-1.

**User Flow 2: Release Coordinator Updates Android feature Flag**

**Scenario:** Release coordinator updates rollout percentage or version constraints for an Android feature flag.

**Endpoint:** PUT /android\_release\_coordinator/feature\_flags/\<feature\_flag\_name\>  
**Access control**: **can\_access\_android\_release\_coordinator\_page (New)**

**Request Body:**

| {   “min\_app\_version”:str   “max\_app\_version”: str    “rollout\_percentage”: int } |
| :---- |

- **Pseudocode**  
  - **Handler:** AndroidFeatureFlagsAdminHandler.put()


1. **Can\_access\_android\_release\_coordinator\_page,** checks if the user has access to android release coordinator page.  
- If not authorized: raise **UnauthorizedUserException**.  
2. Extract feature\_flag\_name from URL path  
3. Parse request body:  
-   **“min\_app\_version”:str**  
-   **“max\_app\_version”: str**  
-    **“rollout\_percentage”: int**  
4. Validate request body:  
- At least one field must be provided.  
- rollout\_percentage must be in  \[0, 100\] if provided.  
5. Call service layer:  
   **android\_feature\_flag\_services.update\_android\_feature\_flag( feature\_flag\_name, min\_app\_version, max\_app\_version, rollout\_percentage )**

6. Return success response (200 OK)

**Service Layer:**  
**Function: android\_feature\_flag\_services.update\_android\_feature\_flag()**

| def update\_android\_feature\_flag(     feature\_flag\_name: str,     min\_app\_version: Optional\[str\] \= None,     max\_app\_version: Optional\[str\] \= None,     rollout\_percentage: Optional\[int\] \= None ) \-\> None:     """Updates an Android feature flag configuration.          Updates the specified Android feature flag's configuration. Only flags     in LIVE state can be updated. Flags in FINAL state are immutable and     will raise a validation error.          Args:         feature\_flag\_name: str. Name of the feature flag to update.         min\_app\_version: Optional\[str\]. Minimum Android app version.             If None, existing value is retained.         max\_app\_version: Optional\[str\]. Maximum Android app version.             If None, existing value is retained.         rollout\_percentage: Optional\[int\]. Rollout percentage \[0-100\].             If None, existing value is retained.          Raises:         FeatureFlagNotFoundException. If feature flag doesn't exist.         ValidationError. If flag is in FINAL state or validation fails.     """  |
| :---- |

**Update Logic:**

**1\. Fetch feature flag from registry:**  
	**feature\_flag \= android\_feature\_flag\_registry.get\_android\_feature\_flag(feature\_flag\_name)**  
	\[May load from storage if not in cache\]  
	\[Datastore: GET \- 1 Call\]  
2\. **Validate flag is mutable:**  
	If feature\_flag.state \== AndroidFeatureFlagState.FINAL:  
		Raise ValidationError(“Cannot update FINAL feature Flag”)

**3\. Update provided fields:**  
	\-  if min\_app\_version is not None:   
		feature\_flag.min\_app\_version \= min\_app\_version   
	\- if max\_app\_version is not None:   
		feature\_flag.max\_app\_version \= max\_app\_version   
	\- if rollout\_percentage is not None:  
		 feature\_flag.rollout\_percentage \= rollout\_percentage

**4.Validate updated feature flag:**  
	feature\_flag.validate()   
**5\. Persist to storage:**  
	**android\_feature\_flag\_registry.update\_feature\_flag(feature\_flag)**  
	\[Datastore: PUT \- 1 Call\]  
6\.**Clear cache for this feature flag:**	 

- **caching\_services.delete\_multi(\[caching\_services.get\_android\_feature\_flag\_memcache\_key(feature\_flag\_name)\])**

**Datastore Calls:**

- **\[SYNC\] GET: 1**  
- **\[SYNC\] PUT: 1**

**User flow 3: Android Release Coordinator views all Android Feature Flags**

**Scenario:** Android release coordinator wants to see all Android feature flags and their current configurations.

**Endpoint: GET /android\_release\_coordinator/feature\_flags**  
**Access control**: **can\_access\_android\_release\_coordinator\_page (New)**

**Pseudocode:**  
**Handler: AndroidFeatureFlagsAdminHandler.get()**

	

1. **Can\_access\_android\_release\_coordinator\_page,** checks if the user has access to android release coordinator page.  
- If not authorized: raise **UnauthorizedUserException**.  
2. Call service layer:  
   **android\_feature\_flag\_services.get\_all\_android\_feature\_flags( )**

3. Convert to dict format  
4. Return JSON response

**Service Layer:**  
**Function: android\_feature\_flag\_services.get\_all\_android\_feature\_flags()**

| def get\_all\_android\_feature\_flags() \-\> List\[AndroidFeatureFlag\]:     """Returns all Android feature flags with their configurations.          Fetches all registered Android feature flags from storage. If a flag     is not present in storage (never configured), returns it with default     values (state=LIVE, rollout\_percentage=0).          Returns:         list(AndroidFeatureFlag). List of all Android feature flag domain objects.     """ |
| :---- |

**Retrieval Logic:**  
 **1\. Fetch all registered Android feature flag names:**  
   feature\_flag\_names \= android\_feature\_flag\_registry.get\_all\_feature\_flag\_names()

**2\. Load from storage:**  
   feature\_flags\_dict \= load\_android\_feature\_flags\_from\_storage(feature\_flag\_names)  
   \[DATASTORE: GET\_MULTI \- 1 call\]

**3\. Process each flag:**  
   result\_list \= \[\]  
     
   for flag\_name in feature\_flag\_names:  
       if flag\_name in feature\_flags\_dict and feature\_flags\_dict\[flag\_name\] is not None:  
            
	 \# Flag exists in storage  
           result\_list.append(feature\_flags\_dict\[flag\_name\])  
       else:  
             
	\# Flag not in storage, create with defaults  
           default\_flag \= AndroidFeatureFlag(  
               name=flag\_name,  
               state=AndroidFeatureFlagState.LIVE,  
               min\_app\_version="0.0.0",  
               max\_app\_version=None,  
               rollout\_percentage=0  
           )  
           result\_list.append(default\_flag)

**4\. Return result\_list**

**Datastore Calls:** \[SYNC\] GET\_MULTI:1

**Android Feature Flag Registry:**

**File:** android\_feature\_flag\_registry.py

- Registry for all Android feature flags.  
- Only registered Android ff can be fetched or updated.  
- Domain objects are always returned in a consistent shape.


**android\_feature\_registry: Dict{str, AndroidFeatureFlag\]**

- **Key: Android feature flag name.**  
- **Value: AndroidFeatureFlag domain object**

The registry is initialized from **a code- defined list of Android feature flags** to prevent accidental creation of unknown flags from storage.

**Methods:**

- **get\_android\_feature\_flag(name: str)-\> AndroidFeatureFlag:**

|    """Returns the AndroidFeatureFlag domain object.      First check memcache for cached value. If not found, loads from storage and caches the result.     Args:         name: str. Name of the Android feature flag.     Returns:         AndroidFeatureFlag. The corresponding domain object.     Raises:         Exception. If the feature flag name is not registered.     """ |
| :---- |

- **update\_feature\_flag(name: str, min\_app\_version: str, max\_app\_version: Optional\[str\[, rollout\_percentage: int,) \-\> None:**

|    """Updates an Android feature flag.     Args:         name: str. Feature flag name.         min\_app\_version: Optional\[str\]. Minimum Android app version.         max\_app\_version: Optional\[str\]. Maximum Android app version.         rollout\_percentage: Optional\[int\]. Percentage rollout \[0–100\].     Raises:         ValidationError. If the flag is FINAL or validation fails.     """ |
| :---- |

- **load\_feature\_flag\_from\_storage(name: str) \-\> Optional\[AndroidFeatureFlag\]:**

|    """Loads Android feature flag config from storage.     Args:         name: str. Feature flag name.     Returns:         AndroidFeatureFlag | None. None if no storage entry exists.     """ |
| :---- |


- **load\_feature\_flags\_from\_storage(name:list\[ str\]) \-\> Dict\[str, Optional\[AndroidFeatureFlag\]\]:**

|    """Loads multiple Android feature flags from storage.     Args:         name: List\[str\]. List of Feature flags name.     Returns:         Dict\[str, Optional\[AndroidFeatureFlag\]\]. Map of name to flag.         Value is None  if flag not found in storage.     """ |
| :---- |


- **get\_all\_feature\_flag\_names() \-\> List\[str\]:**

|    "””Returns all registered Android Feature flag names.     Returns:         List\[str\]. List of all registered feature flag names.     """ |
| :---- |

**Android Platform Parameters**

**Files introduced:**  
**1\. core/domain/android\_platform\_parameter\_domain.py**

* **Domain objects:** AndroidPlatformParameter, AndroidEvaluationContext, AndroidPlatformParameterRule, AndroidPlatformParameterFilter

**2\. core/domain/android\_platform\_parameter\_registry.py**

* **Registry for Android platform parameters**  
* **Methods:** get\_platform\_parameter(), update\_parameter(), create\_parameter()

**3\. core/domain/android\_platform\_parameter\_services.py**

* **Service layer:** evaluate\_all\_platform\_parameters(), update\_platform\_parameter()

**core/controllers/android\_platform\_parameter\_handler.py**

* **Handlers:** AndroidPlatformParameterEvaluationHandler

**User flow 1: Android App requests Platform Parameter evaluation.**

**Scenario:** Android app evaluation of all platform parameters based on its runtime context.  
**Endpoint:** GET /android\_platform\_parameters  
**Access control:** open\_access  
**Request Headers:**

- app\_version\_name: str  
- app\_package\_name: str


**Pseudocode:**  
**Handler:** AndroidPlatformParametersHandler.get()  
	1\. **Extract request headers:**

- app\_version\_name  
- app\_package\_name  
  2\. **Validate required headers are present.**  
  3\. **Parse app\_version\_name into components:**  
- version\_number  
- flavour  
  4\. **Build evaluation context:**  
  	context \= AndroidEvaluationContext(

  app\_version \= version\_number,  
  app\_version\_flavour= flavour  
  )

  5\. **Call service layer:**  
  	Evaluated\_params \= android\_platform\_parameter\_services.evaluate\_all\_platform\_parameters(  
  		context  
  )  
  6\. **Return JSON response.**


**Service Layer:**  
**Function: android\_platform\_parameter\_services.evaluate\_all\_platform\_parameters()**

| def evaluate\_all\_platform\_parameters(     context: AndroidEvaluationContext ) \-\> Dict\[str, Any\]:     """Evaluates all Android platform parameters for given context.          Evaluates all registered Android platform parameters using the provided     evaluation context. Each parameter is evaluated against its rules, and     the first matching rule's value is returned. If no rule matches, the     default value is returned.          Args:         context: AndroidEvaluationContext. Context containing app version             and flavour information.          Returns:         dict. Map of parameter names to their evaluated values:             {                 "param\_1": value1,                 "param\_2": value2,                 ...             }     """ |
| :---- |

**Evaluation Logic:**  
 **1\. Fetch all registered platform parameter names:**  
   	param\_names \= android\_platform\_parameter\_registry.get\_all\_parameter\_names()

**2\. Load parameters from storage:**

- params\_dict \= load\_platform\_parameters\_from\_storage(param\_names)  
- \[DATASTORE: GET\_MULTI \- 1 call\]

**3\. Initialize result dict:**  
   	evaluated\_params \= {}

**4\. For each parameter in params\_dict:**  
     
   evaluated\_value \= parameter.evaluate(context)  
   \# evaluate() method logic:  
   \#   \- Iterate through rules in order  
   \#   \- For each rule, check if all filters match the context  
   \#   \- If a rule matches, return rule.value\_when\_matched  
   \#   \- If no rules match, return parameter.default\_value  
     
  	 evaluated\_params\[parameter.name\] \= evaluated\_value

**5\. Return evaluated\_params**  
**Datastore calls: \[SYNC\] GET\_MULTI:1**

**User flow 2: Release Coordinator Updates Android Platform Parameter.**

**Scenario:** Release coordinator updates rules or default value of an existing platform parameter.  
**Endpoint:** PUT /android\_release\_coordinator/platform\_parameters/\<parameter\_name\>  
**Access control:** can\_access\_android\_release\_coordinator\_page  
**Request Body:**  
	{  
    "commit\_message": "Updated rules for max\_download\_size",  
    "new\_rules": \[  
        {  
            "filters": \[  
                {  
                    "type": "app\_min\_version",  
                    "conditions": \[\["=", "2.0.0"\]\]  
                }  
            \],  
            "value\_when\_matched": 50000000  
        }  
    \],  
    "default\_value": 20000000  
}

**Pseudocode:**  
**Handler:** AndroidPlatformParametersAdminHandler.put()

1\. **Can\_access\_android\_release\_coordinator\_page, checks if the user has access to android release coordinator page.**  
**If not authorized: raise UnauthorizedUserException.**  
**2.Extract parameter\_name from URL.**  
**3\. Parse request body:**

- commit\_message (required)  
- new\_rules (required)  
- default\_value (required )

**4\. Validate request body structure.**  
**5\. Call service layer:**  
**android\_platform\_parameter\_services.update\_platform\_parameter(**  
	**parameter\_name,**  
**new\_rules \= new\_rules,**  
**default\_value,**  
**commiter\_id \= self.user\_id**  
**commit\_message=commit\_message**  
 **)**  
5\. **Return success response (200 OK).**

**Service Layer:**  
**Function: android\_platform\_parameter\_services.update\_platform\_parameters()**

| def update\_platform\_parameter(     name: str,     committer\_id: str,     commit\_message: str,     new\_rules: List\[Dict\[str, Any\]\],     default\_value: AndroidPlatformDataTypes ) \-\> None:     """Updates an Android platform parameter.          Updates the specified platform parameter's rules and default value.     Creates a new version with snapshot history following the Web pattern.          Args:         name: str. Name of the platform parameter.         committer\_id: str. User ID of the committer.         commit\_message: str. Commit message for the update.         new\_rules: List\[Dict\]. New rules as list of dictionaries.         default\_value: AndroidPlatformDataTypes. New default value.          Raises:         Exception. If parameter doesn't exist.         ValidationError. If validation fails.     """  |
| :---- |

**Update Logic:**  
 **1\. Get existing parameter from registry:**  
   	param\_names \= android\_platform\_parameter\_registry.get\_all\_parameter\_names()  
	\[DATASTORE: May trigger GET if not in cache\]  
**2\. Convert new\_rules dict to domain objects:**  
	new\_rule\_objects \= \[ \]  
	for rule\_dict in new\_rules:  
		filter\_objects \= \[  
			AndroidPlatformParameterFilter.from\_dict(f)  
			for f in rule\_dict\[‘filters’\]  
\]  
rule\_obj \= AndroidPlatformParameterRule(  
	filters \= filter\_objects,  
	value\_when\_matched \= rule\_dict\[‘value\_when\_matched’\]  
)  
new\_rule\_objects.append(rule\_obj)

**3\. Create temporary param with new rules for validation:**  
	param\_dict \= param.to\_dict()  
	param\_dict\[‘rules’\] \= new\_rules  
	param\_dict\[‘default\_value’\] \= default\_value  
	updated\_param \= AndroidPlatformParameter.from\_dict(param\_dict)  
	updated\_param.validate() \#Raises exceptions if invalid. 

**4\. Update the registry’s parameter instance:**  
	param.set\_rules(new\_rule\_objects)  
	param.set\_default\_value(default\_value)  
android\_platform\_parameter\_registry.parameter\_registry\[param.name\]= param	

**5\. Update storage model and commit:**  
	model\_instance \= android\_platform\_parameter\_registry.*to*platform\_parameter\_model(param)  
	model\_instance.rules \= \[rule.to\_dict() for rule in param.rules\]  
model\_instance.default\_value \= default\_value  
model\_instance.commit(  
	commiter\_id,  
	commit\_message,  
	\[{  
		‘cmd’: ‘edit\_rules’,  
		‘new\_rules’: new\_rules,  
			‘Default\_value’: default\_value  
		}\]  
)  
	  
**6\. Clear cache**

**Datastore calls: \[SYNC\] GET:1** (If not cached) **, \[SYNC\] PUT:1** (via commit, creates version and snapshot)

**User flow 3: Release Coordinator views all Android Platform Parameters.**

**Scenario:** Release coordinator wants to see all platform parameters and their configuration.  
**Endpoint:** GET /android\_release\_coordinator/platform\_parameters  
**Access control:** can\_access\_android\_release\_coordinator\_page  
**Pseudocode:**  
**Handler:** AndroidPlatformParametersAdminHandler.get()

1. **Call service layer:**  
   all\_params\_dicts \= android\_platform\_parameter\_services.get\_all\_platform\_parameters\_dicts()  
2. **Return JSON response:**  
   {  
   	“platform\_parameters”: all\_param\_dicts  
   }

**Service Layer:**  
**Function: android\_platform\_parameter\_services.get\_all\_platform\_parameters\_dicts()**

| def get\_all\_platform\_parameters\_dicts() \-\> List\[AndroidPlatformParameterDict\]:     """Returns dict representations of all Android platform parameters.          This method is used for providing detailed platform parameters information     to the release-coordinator page.          Returns:         list(dict). A list containing the dict mappings of all fields of the         Android platform parameters.     """ |
| :---- |

**Retrieval Logic (Follow web Pattern):**  
1\. **Get list of all registered Android platform parameters:**  
     
 This comes from the **android\_platform\_parameter\_list.py** file  which defines **ALL\_ANDROID\_PLATFORM\_PARAMS\_LIST**  
   all\_params\_list \= android\_platform\_parameter\_list.ALL\_ANDROID\_PLATFORM\_PARAMS\_LIST

2\. **For each parameter in the list, get from registry and convert to dict:**  
   result\_list \= \[\]  
   for param\_name\_enum in all\_params\_list:  
       param \= android\_platform\_parameter\_registry.get\_platform\_parameter(  
           param\_name\_enum.value  
       )-  
       result\_list.append(param.to\_dict())  
     
   \[DATASTORE: May trigger multiple GETs if not all cached\]

3\. **Return result\_list**

**Registry Layer Functions:**  
**android\_platform\_parameter\_registry.py** (follow the web registry class pattern):

1\.  **get\_platform\_parameter(cls, name: str) \-\> AndroidPlatformParameter:**

| def get\_platform\_parameter(cls, name: str) \-\> AndroidPlatformParameter:         """Returns the instance of the specified platform parameter.                  Checks cache first, then storage, then in-memory registry.         Caches the result before returning.                  Args:             name: str. The name of the platform parameter.                  Returns:             AndroidPlatformParameter. The parameter instance.                  Raises:             Exception. If parameter name is not found.         """ |
| :---- |

2\. def update\_platform\_parameter(  
        cls,  
        name: str,  
        committer\_id: str,  
        commit\_message: str,  
        new\_rules: List\[AndroidPlatformParameterRule\],  
        default\_value: AndroidPlatformDataTypes  
    ) \-\> None:

| def update\_platform\_parameter(         cls,         name: str,         committer\_id: str,         commit\_message: str,         new\_rules: List\[AndroidPlatformParameterRule\],         default\_value: AndroidPlatformDataTypes     ) \-\> None:         """Updates the platform parameter with new rules.                  Args:             name: str. The name of the platform parameter to update.             committer\_id: str. ID of the committer.             commit\_message: str. The commit message.             new\_rules: list(AndroidPlatformParameterRule). New rules.             default\_value: AndroidPlatformDataTypes. New default value.         """ |
| :---- |

3. \_to\_platform\_parameter\_model():

| def \_to\_platform\_parameter\_model(         cls,          param: AndroidPlatformParameter     ) \-\> AndroidPlatformParameterModel:         """Returns the storage model for the given domain object.                  Creates new model if it doesn't exist.                  Args:             param: AndroidPlatformParameter. The parameter domain object.                  Returns:             AndroidPlatformParameterModel. The corresponding storage model.         """ |
| :---- |

### **\[Web only\]** Web frontend changes 

1. We will change the current release coordinator page and URL path with having web as prefix in it.  
2. A new Android Release coordinator dashboard, parallel to the existing Web Release Coordinator UI.  
- **URL**: /android-release-coordinator  
- **Visibility is controlled via**: GET /can\_access\_android\_release\_coordinator\_page   
- Web Release coordinators can view the page but cannot edit Android Configuration.

**User Flow 1: Ensure only authorized users can access Android Release coordinator dashboard**  
	\- On navigation attempt, frontend calls:  
		GET /can\_access\_android\_release\_coordinator\_page   
\- If unauthorized:  
		\- Redirect to 404 Page  
\- If authorized:  
		\- Load Android Release Coordinator dashboard. 

**User Flow 2: View Android Feature Flags**  
		GET /android\_release\_coordinator/feature\_flags  
\- Same UI and components as Web Release Coordinator.  
\- [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) See for more info.

**User Flow 3: Edit Android Feature Flags(LIVE only)**  
\- **Editable Fields:**  
		\- Min app version  
	\- Max app version  
	\- Rollout percentage  
\- **Frontend Validation (before save enabled):**  
		\- min\_version \<= max\_version (if max defined)  
\- min\_ version \>= hardcoded\_min\_version  
\- rollout\_percentage between \[0, 100\]  
\- **UI Feedback:**  
		\- save button disabled on invalid input.  
\- Error icon shown on Save button.  
\- Tooltip on hover explains the validation error.  
**Save Action:**  
	PUT /android\_release\_coordinator/feature\_flags/\<feature\_flag\_name\>    
\- Success toast shown.  
 \- Updated values reflected immediately. 

**User Flow 4: View Android Platform parameters**  
		GET /android\_release\_coordinator/platform\_parameters  
\- Same UI and components as Web Release Coordinator.  
\-  [Mocks: Remote Configuration of Android Feature Flags & Platform Parameters via Web](https://docs.google.com/document/d/1hyRD-QbD1Yzw6L_DAr_pISTKXNBXCqrHPfshCX8oLRA/edit?tab=t.0) See for more info.

**User Flow 5: Edit Android Platform parameters**  
\- **Editable Fields:**  
**\- Default Value**  
**\- Rule list:**   
	\- Min app version  
	\- Max app version  
	\- Flavour  
\- **Frontend Validation (before save enabled):**  
		\- min\_version \<= max\_version (if max defined)  
\- min\_ version \>= hardcoded\_min\_version  
\- **Save Flow:**  
	\- User clicks Save.  
\- Confirmation modal appears.  
\- Commit message required.  
\- On confirmation:  
	PUT /android\_release\_coordinator/platform\_parameters/\<parameter\_name\>  
		\- Success toast shown.  
 \- Updated values reflected immediately.   
\- **UI Feedback:**  
		\- save button disabled on invalid input.  
\- Error icon shown on Save button.  
\- Tooltip on hover explains the validation error.

**Frontend Services:**

1. domain/android-feature-flag/android-feature-flag-backend-api.service.ts

| async getAndroidFeatureFlags(): Promise\<AndroidFeatureFlag\[\]\> async updateAndroidFeatureFlag(   name: string,   minAppVersion: string | null,   maxAppVersion: string | null,   rolloutPercentage: number ): Promise\<void\> |
| :---- |

2. domain/android-platform-parameter/[android-platform-parameter-backend-api.service.ts](http://android-platform-parameter-backend-api.service.ts)

		

| async getAndroidPlatformParameters(): Promise\<AndroidPlatformParameter\[\]\>async updateAndroidPlatformParameter(     name: string,     commitMessage: string,     rules: AndroidPlatformParameterRuleBackendDict\[\],     defaultValue: PlatformParameterValue   ): Promise\<void\> |
| :---- |

**Frontend Model:**

1. domain/android-feature-flag/android-feature-flag.model.ts

   Model Structure:

| export interface AndroidFeatureFlagBackendDict {   name: string;   description: string;   state: 'LIVE' | 'FINAL';   min\_app\_version: string;   max\_app\_version: string | null;   rollout\_percentage: number;   last\_updated: string | null; } |
| :---- |

   

		Frontend Domain Model:  
		

| export class AndroidFeatureFlag {   constructor(     readonly name: string,     readonly description: string,     readonly state: 'LIVE' | 'FINAL',     minAppVersion: string,     maxAppVersion: string | null,     rolloutPercentage: number,     lastUpdated: string | null   ) {}   static createFromBackendDict(     backendDict: AndroidFeatureFlagBackendDict   ): AndroidFeatureFlag; }  |
| :---- |

2.domain/android-platform-parameter/android-platform-parameter.model.ts

	Model Structure:

| export interface AndroidPlatformParameterBackendDict {       name: string;       description: string;       data\_type: string;       rules: AndroidPlatformParameterRuleBackendDict\[\];       rule\_schema\_version: number;       default\_value: PlatformParameterValue; } |
| :---- |

Frontend Domain Model:  
	

| export class AndroidPlatformParameter {   constructor(     readonly name: string,     readonly description: string,     readonly dataType: string,     public rules: AndroidPlatformParameterRule\[\],     readonly ruleSchemaVersion: number,     public defaultValue: PlatformParameterValue   ) {}   static createFromBackendDict(     backendDict: AndroidPlatformParameterBackendDict   ): AndroidPlatformParameter |
| :---- |

### Documentation changes

- *Add 2 new wiki pages for introducing Android feature flags and platform parameters in Oppia Web.*  
- *Update existing wikis for release coordinator, feature flags and platform parameters to have web as prefix before them.*  
  *Few such wikis:*  
  - [*https://github.com/oppia/oppia/wiki/Launching-new-features\#follow-the-steps-below-to-add-a-new-feature-flag*](https://github.com/oppia/oppia/wiki/Launching-new-features#follow-the-steps-below-to-add-a-new-feature-flag)

# Implementation Plan

### Milestone Table

| *What are the independent pieces of the project and what happens in each? Fill in the following planning table, where each row represents either a PR or an action (like “flip this flag on the server”). Try to break down the work as much as possible so that the individual PRs are small \-- but make sure that each PR can be merged into the develop branch without causing any breakages. In other words, the develop branch should always be release-able. The “description of PR / action” should start with a verb, and clearly indicate: What is done in the PR What the PR accomplishes, i.e. what an end user can do after the PR is merged.  For example: "Create a new unpublishing message modal in the topic viewer page. After this, learners who visit the topic will be notified that the lesson has been unpublished."  The “Prerequisite steps” column should indicate which PRs must be merged, or which actions must be completed, before the corresponding row can be started. Use the serial numbers in the leftmost column of the table. Important notes:  Any UI PRs should concurrently handle desktop/mobile, accessibility, i18n and/or “dark mode” (rather than as separate follow-up PRs). End-to-end/acceptance tests should be written as part of the PR which introduces the changes they are testing. If this feature affects or requires changes to existing data on the production server: please (a) call this out explicitly, and (b) list the “action” steps that need to be taken in order to release this feature safely, including details of any prerequisites that need to be ensured before launch, migration jobs that need to be run, or flags that need to be flipped.  If any changes need to be communicated to users or developers (e.g. via email): ensure that these steps are included, and include the text of the communication below the table. If your feature falls under the [Web launch process](https://github.com/oppia/oppia/wiki/Launching-new-features#how-to-use-feature-flags), clearly explain which PR sets the feature flags to the relevant stages.  Also, include an “action” row for submitting the feature for feature testing. You might also want to include a “demo to PM/users” at the end of any significant milestone, such as when a certain chunk of functionality is feature-complete. This is a good chance to get feedback from project stakeholders. You can add “annotation rows” to summarize the current status of key functionality at relevant checkpoints. See example in the table below.* |
| :---- |

| No. | Description of PR / action | Prerequisite steps | Target date |  |
| :---- | :---- | :---- | ----- | :---- |
|  |  |  | **PR creation** | **PR merge** |
| 1 | *(Example)*  PR: Add new feature flag XYZ\_NAME (in DEV stage) to gate feature ABC. | *(Example)*  N/A |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 | *(Example)*  PR: Create the UI for ABC. Since this is the last PR for feature ABC, also set the feature flag XYZ\_NAME to the TEST stage. | *(Example)*  1.2, 1.3 |  |  |
| – | *(Example of “annotation” row – use this when you have completed a key subset of the milestone requirements that can be demoed.)* **Note**: At this point, feature ABC is fully implemented. User journeys D and E can be fully tested by turning on the feature flag XYZ\_NAME. |  |  |  |
| 5 | *(Example of “demo” row. The date on the right should be the planned demo date.)*  Demo to PMs/users: *\[describe which functionality will be demoed\]*  | *(Example)*  1.4 |  |  |
| 6 | *(Example of an “Action” row. The date on the right should be the expected date for all actions here to be completed.)* Run the feature testing process for feature ABC on the backup server following this [release testing doc](https://docs.google.com/document/d/1Xg0MSIpYUEax8dqH39CKhgqt30f-kr8wxO2F4aSxwz4/edit?tab=t.0)[^2].  | *(Example)*  1.4 |  |  |
| 7 | *(Example)*  PR: Set feature flag XYZ\_NAME to PROD stage, so that it is launched in the next release. | *(Example)*  1.6 |  |  |
| ... |  |  |  |  |

# Feature flags

| *If this feature includes user-facing changes, you will need to gate them behind feature flags. See the [”Launching New Features” wiki page](https://github.com/oppia/oppia/wiki/Launching-new-features) for instructions on how this works. If your project involves multiple feature flags, ensure they are independent – feature flags should not depend on each other. Any user-facing changes should go through feature testing before the corresponding feature flag is turned on. This is because even simple changes will have non-trivial implications for accessibility, internationalization, and multi-screen/form factor support. In “Needs sync between Android / Web?”,  enter “Yes” if the flag should be turned on for Android/Web at the same time, and “No” otherwise. (Some features will require a coordinated launch between the Android and Web platforms, to prevent the Android app experience regressing when the flag is turned on for Web.) In the “Key Stakeholders” column, list the teams who should attend the feature testing demo session. This should include other affected Oppia teams, e.g. the “other” Web/Android platform team, dev workflow, language accessibility, GTM, partnerships, and so on.* |
| :---- |

| \# | Name of feature flag | What happens when the flag is turned on | Needs sync  between Android / Web? | Key stakeholders |
| :---- | :---- | :---- | :---- | :---- |
| 1\. | AndroidPlatformParameter |  | No |  |
| 2\. | AndroidFeatureFlag |  | No |  |
| 3\. |  |  |  |  |

### Future Work

| *List here any future technical work that isn’t in the scope of the current implementation, but that should be taken up after the current project is complete. Note that the full product design should still be done upfront \-- but, if you’re explicitly deferring a particular part of the technical implementation to a separate phase, make sure to add that to this “Future Work” section, so that it’s clear what the implementation scope of the doc is. If you need help, please feel free to ask the Oppia tech leads for advice about scoping. Note: For future work that we definitely want to do, please file issues on GitHub, and link to those issues from this document.*  |
| :---- |

...

| *You’ve reached the end\! Please get a final approval from reviewers. Do not start implementation until you have all the approvals.* |
| :---: |

## Reviewers will verify:

* Are the architectural decision analysis solid, and are the conclusions supported by the data? Are there other key architectural decisions that should be considered (but haven’t been)?  
* Will the proposed approach scale?  
* Can the overall implementation be simplified?  
* Are there any potential red flags or risks (particularly around security) in the proposed solution that need further investigation?  
* Are all docstrings and function/class names clear and correct? Are the storage models, domain objects, services, functions, etc. named appropriately? (These will be used during implementation.) Are there any naming collisions with already-existing concepts or names in the codebase? (We should avoid these.)   
* Is there a clear description of how errors are handled, and is this description correct and in line with what we currently do in the codebase?  
* Feature completeness:  
  * Are all aspects of the feature addressed by the proposed implementation?   
  * Does the proposed solution satisfy the technical requirements and constraints in section 1?  
* Storage models:  
  * Are the operations on storage models efficient?  
  * Is the storage model ID generated correctly?  
  * Are the deletion/export policies correct?  
  * Are any new models added? If so, has adequate consideration been given to backfilling data?  
* Does the launch plan make sense? Is each milestone sufficiently broken down? Are the milestones in the correct order?  
* Is there any future work that is sufficiently definite/important that we don’t want to lose track of it? If so, has a GitHub issue been filed?

[^1]:  You can create a new “bookmark” in Google Docs by putting your cursor on the relevant section, and then selecting “Insert \> Bookmark” from the document menu.

[^2]:  This is a sample; you should make a copy of this template and populate it for your specific project.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,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/cx/FL3+5MkTlgJJkyal8lmzZlXJlNfVW3cMTYmm1zkuV64cVeK58PcTAxZAdCLPSBPhdc5QbRKQrFq1arSYeF3QCPG6IAgvizNnzlSuXJkFAQbF4uaSxUZWrVr13XffIXtfX1/EXKNGDWsKQfg/hnhdEISXxePHj5s1a8Zum83ugAEDLl26ZE3hNvfv31+6dGmaNGleeeUV1g3qxyAE4f8y4nVBEARB8BzE64IgCILgOYjXBUEQBMFzEK8LgiAIgucgXhcEQRAEz0G8LgiCIAieg3hdEARBEDwH8bogCIIgeA7idUEQBEHwHMTrgiAIguA5vNDrxeMBtWvXtlbrn7BmEQ+YMWOGtZaxhbUqurFjxw7zWW7evGm9LMRjLH+n/bfffrOmEOIZzv3l0D96/F+gY8eOll5TvNDrJ0+etJ6KdT788EPrqX9i48aN1lNxTZ8+faynYot+/fpZT+lDvXr1lixZYn68ePFi5syZna4L8Zr333/f+WPz5s2dPwrxjf/3//6f5cyCBQssZ4T4RvEIqzGFeP2lI16PGeJ1rRGv64V4XUfE63GGeD1miNe1RryuF+J1HRGvxxni9ZghXtca8bpeiNd1RLweZ4jXY4Z4XWvE63ohXtcR8XqcIV6PGeJ1rRGv64V4XUfE63GGeD1miNe1RryuF+J1HRGvxxni9ZghXtca8bpeiNd1xGavh4aGrlix4tNPP82WLVuaNGkWLlzImfnz5//yyy8///wzl54+fWq9JwIHDhxYvny59awTtnt98+bN6dKly5gx40cffVSiRImwsDBripdAvPU6j3/48OGvvvoqa9asH3zwweDBg589e2ZN5IqZM2f6+PhYz0aA7iPDd95559ixY9ZrUeAlef2LL77o0qWL9Wx4hg4dytjes2eP9YIrypcvbz0VIxYvXsxUom179+69fv166+WXD48cEhJiPRtTXpLXX3vtNeup8AQFBRUuXDhLlix37961XovAb7/99o+DISo8ePCA4crQunz58ueff269HE2ePHlCgLKejT5k8ujRI+vZF/CSvN6oUaPAwEDr2fAUKlQoWbJkpLReiIC/v3/37t3Pnz9vvRB9atWqlTRpUoouUKDA48ePrZdjBDP36tWr1rMvE5u9vnLlyly5cqFwYh9faZp169Ypr584ceLatWto3npPBAhh8+bNs551wnav05c9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>

[image4]: <data:image/png;base64,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4QVZ/k8//XT27NmTJ08SHGzYsIGazsZ9SvcYS+nuUUp306V0N11Kd08Suvfu3Vt+lMWme3wIunv/5p3SPYZSunuU0t10Kd1Nl9Ldk6D7Qw89ZP+g6iOPPPKfhK2kSZO656CKTkp3j1K6my6lu+lSunsSdH/vvfeSJUt2H3L3OJHm7jGU0t2jlO6mS+luupTuniTvzJ89ezZFihRKd3+S0t2jlO6mS+luupTunmR/qy44ONj5nfkEK6V7DKV09yilu+lSupsupbsnOf+9e58+fZTufiOlu0cp3U2X0t10Kd09yfV/1SV8dib8ESYQKd09SuluupTupkvp7kl3+p9oE6yU7jGU0t2jlO6mS+luupTu////kmsdWzVs2PCll15yl96L3KO5swCz++F7V+3atd1F9662bdu6B/eA1LlzZ/fg4kiffvqpuyiO5By/+54f6eGHH3YX+Zf83o4BAQHuIn+R39tOVKtWLXeRX8hpPlvR0/3JJ590F90vDRs2zF10Zy1evNhd9ID06KOPuosekJ555hl3UcKWK6P97LPPnC9Vpug///mP86VZb+D9j+upp55yvvz444+dL1UJXJcuXXIXWVK6x42U7rGW0t0/pHQ3V0p3o6V0j18p3WMtpbt/SOlurpTuRkvpHr9SusdaSnf/kNLdXCndjZbSPX6ldI+1lO7+IaW7uVK6Gy2le/xK6R5rKd39Q0p3c6V0N1pK9/iV0j3WUrr7h5Tu5krpbrSU7vErpXuspXT3DyndzZXS3WjFJd3XrFkDQt5444233norb968kZGRuXLlypYt2/jx42OC261bt06dOtVdektxSPcxY8awa995553nnnuuXr167ttxKlPojrGKFCmC7d5+++2AgIB169a5a0TRtm3bpkyZ4i69XXXr1g0LC2OpIyIi3Pf+SfFBd8Y8YsQId6lDpUqVOn/+vP2SCS5YsMBx/2/NnTv3tddei8n/b8X+dxfdo1jA5MmTc6AyZcp048YN9+14k8uzx1rxQfcdO3YMHTrUXerQl19+mTp1avvljBkz0qVL57j/tzDuq6++umfPHveNKMqRI4e76B6FHVlSDtd7773XqFEj9+0Y68qVK4ULF3aXxliXL1/+/PPPQ0JC3DeiU3zQndVOmTKlu/R2vfDCC/b1zZs306ZN67j5txYtWpQqVapPPvkE3+W+d7s+/fRTfL679F4ktgNne/furV+/vvv2PYoZ4WzdpfGguKR77ty5AXloaOjFixc5WsePHxe6U3L16lV37SiaP3/+hAkT3KW3FId055wHBgZisMOHDydJksR9O05lCt137tyJyTjzYruCBQu6a0QRnnHcuHHu0ttlHN1Pnz7tHOqd6N6jR4/ffvuNU+q+EUX/6Mj+UZwjvPm1a9fuM92PHDniLoqVHgjdg4KCjh49ar+8E9179+7966+/xsSOZALuonvUiRMnOFYchwdLdybLJo/JlH0Pju5O292J7v369SONZD3/ke7ECnffLf+okydP5s+fn744jN7pjodhD7hL40FxRvcDBw4UL17cfrlv375Dhw45c3c2JfAm4ylQoMD169cpwVtVq1aNZHHevHljx4598803OUJnzpxxtPpfxSHd6YWTJtd07bM20/fff8/5/+qrr3Cg7du35/r1119v2LBh8+bNpWb16tX//PPP33//nQSXWwIJIscSJUo4swSXjKA7liIy279/v7wkJZ02bRqHioCXdciTJ4+8p5I5c2YWAXtRggXTp0/PShYrVuznn39u27btyy+/zK7dvXt3hgwZOHWbNm3y3U53FrZkyZJsgEGDBnEgz507V6hQoZdeegluOQdjK+Z0p4svvvgiTZo0JCVLly6lBBuR7GbMmLFNmza8ZFSMgSztxx9/xHAdOnQYOHAgRmTk69ev54JnW7ZsySAldw8PD2cnMFSsH5XurMzTTz/NyImEqInLpjtZIl4yZRaNjcGGZxkfeugh1pYLKMKs6YJhULNKlSqdOnXidFBCokkLZIe4XVdfq1evzpIlS7Jkyfr37y90J/di/RkwtmAktNm9e3fa5IAwZtfjPutg8iBGYWDUDw4OzpkzJ2etaNGijHbixInffvstFWrVqsXqUR/L9urVa9myZXh2Gh88eDD1sTULxeOsJ9vg3XffJU7yWQeHl6ztxo0b3R3fUszpjh1ZCqbGAnLQKMEEWAo7MkifBYbGjRszWVYDf82Z5Rrz4T02bNjAGr7yyistWrSQJeVUslysM3VatWoVLd1TpEiBHVkTaoodiedYAV7Wq1ePRzAKm4Tg4OGHH8abzZw5k/qsA11wl72BB+vcuTPYo2Tt2rUsbPbs2f/66y9XR+vWrfvoo4+wY9++fYXu2JEtymRpgWCFpe7ZsydDHTJkCDNyPY5I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>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAo0AAAEGCAIAAACPUDicAABh4UlEQVR4XuydebxNVf+AhcSreptL3kpSlFRUSiRChkxljEjmTBkj80zIPIXMySzzPGSe5zHzNVyua/aa7nX37/nt72u37XuvDtbVsfs+f5zPPmvvvfYav89a5xxuPEtRFEVRlGAlnjdBURRFUZSgQT2tKIqiKMGLelpRFEVRghf1tKIoiqIEL+ppRVEURQle1NOKoiiKEryop5VgYcuWLadOnfKm2pw9e9abdItcuHBBXtesWeM9d4tcvHhx3bp1R44ckbenT5+WPC9durTGhuMrV67E+CASr1275rwlE7llw4YNu3fvJuXo0aO0w583uDh37pwc7N+/nwJERETceD4g9uzZI09cu3bt4cOHvacNQdmuXr3qTY1GaGgoFaHiUVFR3nMxQcvv2rWLtjp+/HiMzcsFpG/atEneciVv6Ysbr7oBGjN6g9NH3BgZGelJV5S/BfW0EiykSZNmwoQJ3lRbXQMHDvSm3gpoo2PHjhwgp3jx7nTM//zzz4kSJWrWrJm8nTt3ruS5cePGeDYc79ixI8YHkejWRqtWreSWBx988K233jp58mS7du1oB9cd/4MMJ0+ebNkKyZo16wMPPHB7a5eiRYvKExMmTJg/f/6bO+z2oGBjxowJCQnxnrgRljXFixenJTNkyPDHH394T8fEkCFD0qZNS1sNGDAgxubdunUr6Y8//rgsy5544gne3nw5Ur58+dSpU3sS8T03SiaK8rcTw1hXlL+F2Dxdq1atSpUqeVNvhX379klYJ/4Syr2nb5F06dI1bdr0v//9r7wNDw+XzLt27XrfffdxzKm2bdsmSJDghttsons6VapUbChnzZr16KOPVqxYMTZP58mTp1evXhywlSQTNvHeKwIDT5csWZInbt68+V//+tf48eO9V9wxP/zwAyVk4+494YIeefbZZ/Ply7dt27bq1as/8sgjgWxeWcpI3SHGfhRPQ4cOHVjQ0AXqacUHqKeVYEE8TeCOHz/+Y4899swzz0isTGSzdOnSq1ev/tuGvSB2RN5c8NBDD3322WfDhg1ji8kxKfPmzTt37hwHDz/8MK/16tXLli0bB8jA2U9jhVdeeQVREcoHDx5MSokSJZ566inkwQV79+51SsW2L5693+X1nXfemTp1KsVLkiTJnDlz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q55WlABRT/sb9bQx1NN3yIgRI5x/lGWppxUlYNTT/kY9bQz1tFnU04oSIOppf6OeNoZ62izqaUUJEPW0v1FPG0M9bRb1tKIEiHra36injaGeNot6WlECRD3tb9TTxlBPmyXYPH3t2rUrV654U28kPDz8L4t94cKFM2fOREVFeU8oyu2invY36mljqKfN8pfCM8uiRYv69et36dKlY8eODRs2bObMmadPnx43btyoUaM4y3FISAizZceOHT179oyIiEDbgwYNcv+BLxLTpUu3Z88eDriMfLp37z5w4MAjR45wL8fnz5/nspo1ax44cOD48ePOjYpyh6in/Y162hjqabPcZU/nzJkTlW7ZsqV06dKNGjX65ptvdu3a1bBhwzx58rD3nTNnDnJdt25dyZIl+/btu2nTptDQ0Hbt2pUoUcLJITIy8sMPP9y+fTuyR/Dr169PnDjxxIkTub5IkSIDBgzA/VyD+MmQNYHr4eY5deqUN0nxL+ppf6OeNoZ62ix32dMI9eGHHz548GDChAmJeg899NDatWsvX76McSnJ66+/fubMGTxdtWpVuR6Xc9mDDz7ozqRUqVKHDx92PJ0gQQIS69Spkzt3brmA3TZnOUiWLJn7RuOwDujVqxc7e+8JxY+op/2NetoY6mmz3GVPZ8+efeDAgQsXLixevHjz5s2rV6/Ozli+kH7qqadmzZolnma3/dNPP7GfxsedOnXiYncmePrq1as1a9bkMrenixUr9vPPP7OfRtJbt24lsVChQu4bjYOn48WLlyhRIt1Y/xNQT/sb9bQx1NNmucueXrJkSc+ePS9cuHD06NFhw4aNGTPm9OnTkZGRnOrcuTOFYW8dGhq6c+fO3r17s0/lFF6fOnWqO5PJkydbtiOnTJly7NixH374gbfz5s1D6pI5bwcMGMC9Y8eOdd9oHPE0tG7d2ntO8R3qaX+jnjaGetosd9nTt0226+TIkcN7LibwdPv27eP6996Op0GWC4qPUU/7G/W0MdTTZrlXPB2cuD3dp08f72nFX6in/Y162hjqabOop+8Ex9PJkyc/e/as97TiL9TT/kY9bQz1tFnU03eCePqBBx6QL8UVf6Oe9jfqaWOop82inr4T8DSS/vXXX70nFD+invY36mljqKfNop6+EyZPnqw76X8O6ml/o542hnraLOppRQkQ9bS/UU8bQz1tFvW0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EBUVNWXKlEmTJpG+ZcuWPn36pEuXrkePHrVr12YnPXTo0E8++SQsLCxz5swrVqwYPny4eloJQtTTxlBPm0U9HRdcvXp1zZo13tRgZcyYMbwOGjSoS5cu77//Psc4+9FHH42MjBwyZAivefPmPXToEDonXTydKVMmLmvRooXkgKcnT5587ty5J598krfNmjXLmDEjB5he/ipi165dhw0bliFDhoiIiJIlS6qnlSBEPW0M9bRZ1NPGOX369P3338/e0XsiWFmyZMnevXvbtWs3fvz4NGnSXLp0iW30Y489du3aNRTOa758+XLmzMkBwhZPZ82a1bJ9LDmIp69cueJ4mn05eY4ePXrkyJEHDhxo1KjRrFmz3nnnHa6ZOXOmeloJQtTTxlBPm0U9bRYk/cQTT8SLF+8e8jQsXbp07dq1V69ePXXq1IIFCxgVvGLf0NBQXtetW7dv3z6G9ObNm1etWnXhwoXVq1dz1+7du+V2rjl+/DgiX7hwoaTTDitWrNiwYQOnJHP25U7mrAxcD79nUE/7G/W0MdTTZlFPm6VLly7yRxruLU8rgaCe9jfqaWOop82injZIv379rv/Nw3ivv/76fMVfJE2a1Nvlio9QTxtDPW0W9bRBQkNDEyZMqPtpv6L7aX+jnjaGetos6mmzVKlSRT3tV9TT/kY9bQz1tFnU08YpXry4etqXqKf9jXraGOpps6injXPw4MEECRKop/2HetrfqKeNoZ42i3o6Ljh06NCQIUO8qco9jnra36injaGeNot6WlECRD3tb9TTxlBPm0U9rSgBop72N+ppY6inzaKeVpQAUU/7G/W0MdTTZlFPK0qAqKf9jXraGOpps9xNT5cpU4bX3Llzp06d+syZMxxv3bo1ZcqUBQsWvHLlSuLEiatUqUJi9erVPTcqSjCgnvY36mljqKfN4ng6Kirq2LFjN540DJ7mKX379r148eKECRMuX74sSkbSlSpVwtOtW7detWqV4+l9+/Y9/vjjP/74I2dr1arF29q1azdq1GjevHkffvhhd5vOnTuXL1++atWqu3fvbtGiRbly5fbs2VO6dGnGyY4dO9q0aVO/fv2ZM2feUA5FuS3U0/5GPW0M9bRZHE9v27atbt26N540jOynGzZsWLly5ePHj586dapLly5yqlixYnh6yZIl1W0kETE/8sgjCHjcuHHbt28fNGjQ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BnAw4+lg4CaevjvcqqeDh8A9fYcQ7KpVq+ZNjWOc35EpgXv6DiEkSSC+m7i/3vqnEbin7xD5iMKbGsegW2KUN/WehbVIpA1LQ++5WFBPG0M97YGx9e6NsAGVb4DijtWrV3se2qlTp+g/h7lrsAFyFybGjeBdI448Lf+yyKFfv35t27Z1vmyOI8qVK+d+aOnSpZ0fr919Zs+e7S7Mu3ddKnHkafnMxs2oUaNu8otUU3geytbf+fHH3ady5cruwhQrVsx7xS1y7NixMmXKVK1aNbbPvaKjnjaGetrDtWvX9txIXMduyx7QnocGPhnigrCwME95vFfcReLI03Sru4InT568C748dOiQ+6EhISHeK+4i586dcxdmz13v5Tjy9NWrVz31uvMfEgWC56He03eXw4cPuwtz8OBB7xVxj3raGOppJciJI08rfztx5GklSFBPG0M9rQQ56mm/op72N+ppY6inlSBHPe1X1NP+Rj1tDPW0EuSop/2KetrfqKeNoZ5Wghz1tF9RT/ubW/B0/vz5V9x1vIWInU6dOnlvvrvE6Ol58+Z5rws+XnvtNben/5aOVu4C7vCdKFEi72nlnsXj6caNG3uvUO5lbsHTiqIoiqIEA+ppRVEURQle1NOKoiiKEryopxVFURQleFFPK4qiKErwop5WFEVRlOBFPa0oiqLce9ydv9sRHh4eEhJy/Phx56/F333U04qiKMq9x2uvveZNigO6d+9euHDhffv2qacVRVGUfwRfffXViBEj5s6dGxUVValSpcWLFydPnvzSpUtVqlQpV64ciY888givCJKLa9SoUb9+/fbt22/fvr1mzZr9+vWbMmXKsGHDuB1Pjx07tkSJEmysU6VKNXTo0KVLlxYsWHDatGncwr05cuQYPnw4r+3atZP/36lu3bpdunSZPXv2559/fubMmQIFCqxatapMmTKRkZHffPNNoUKFkDE5TJ06tVGjRpzq2bPns88+i6ebNGly+vTppk2bVqtWjYMTJ060atXq7bffjoiISJEixaxZs86ePdu5c+ciRYqQAylUKlmyZAsXLqxYseK2bdsmTZr0/vvvS/W5IE+ePM2aNaPYPJHSlixZkl17rVq1ihUrRu0o3rx581wNpp5WFEVR7iJYGaX16tXr8uXLOGnUqFFp0qRZv359p06dnnvuOfSZOnVqTq1du5aLUR2G++6770aOHJkzZ05EeP78+ZMnT5Ii++nixYvjbGx64cKFsLCwl156iQvQNmLG+pZta7encSQXNG7c+NSpUzxdcvj99995LmJGkNiXC5555pkOHTpcvXoVMYunjx49+t///nfmzJnHjx9nTXDlypWuXbviaeRKJqGhoRcvXmRZgIaxLylZsmThlWXE5MmTUfLq1aul+jt27Dh48P/aO2PVxIIoDD+QVQoRFVECphAbwUYRBQvXTiwEKxsfQAQTRMFaEex8AgsLCxFt7FxRsFAEJWDlzcc9EBbbZYMs/1dc5s45M2du9c1JIPm93+8Hg4H9telkMon4SajX62zL1/V6PUs25GkhhBA/R6FQoEuOx+OVSoXXUqk0Ho/f3t7m8/kvF1pMCxnFYhFNoj3ESbN7OBxQJhY3T9OR87pYLFqt1sfHh8fjwaOj0QgxU8VxPU07jkQd19MUImEymeDp9XrtuJ6m30WQm82GTt3v95MwHA4pSp+NjM3T0+mUFp8EDkBTTqjf7+NpRM4mNNDtdrvZbOLpTCbDzOvrq+M6GCWz9uXlxT5nNptRmmsBNw+KOu7/WaAi8rabAdFut2vJhjwthBDi56B3bDQaKJMm2HE1jNu8Xi/mQ1c0xExaKJ/P88Tf9NzID/Uul8tgMIgCt9stnqaXRZyXy6VcLofD4dVqhTtDoRDid9xfLcdiMTzNvM/nS6VSFKVxDwQCtVoNU357+n6/R6PRbDbLSeiV2YHGmu48Eonkcjnz9G63Y55rBM0xJ2GcSCS+PU2UA3AzoK1/8DTeZR/SOCchatFwsydXB47H59A9H49HeVoIIcRTgKjw3Ofn5/l85hWxMWM/AWaAPhlYCLHxxN/0naQ57i930RhRy2SA2JhnQ+aJIk4GLGGSVSTgaVtFCYoyPp1O5LODpV2vV57sZicxUxJizHKWcAyevDJPDiGsTOj9/Z1JO5hFUS8D29C+iLGFrLR9EU+LMuAwVCTEtrac8e12c/5AnhZCCPHfUq1WH6f+GsyaTqc7nc5j4N8gTwshhBDPizwthBBCPC/ytBBCCPG8fAH/aLnORWM0pQAAAABJRU5ErkJggg==>

[image6]: <data:image/png;base64,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21dIwgNScwF9hx6QmlnyJM16wIOqkD9lxYoV9l2K2+nTp499l+K1bNy40b5LcRHbt2+371IPSB71gJSiHpAOUQ9wE+oB7kY9IMWoB6QU9YB0iHqAm1APcDfqASlGPSClqAekQ9QD3IR6gLtRD0gx6gEpRT0gHaIe4CbUA9yNekCKUQ9IKeoB6RD1ADehHuBu1ANSjHpASlEPSIeoB7gJ9QB344AH+Pr6ZsyYMVOmTLly5UpISDh58uRvv/32+eefx8fH20/1IDg4OCgoyL7X5N49oHfv3m+88cZrr702e/Zs+7F7xhs9oF27dh999BGNUrBgQdrIfvhWat36h+A2atasGRcXl6J/n+KUB0RFRb366qv2vTeZNGmS9e+PKONnn31263E7ZcqUoSCee44cOSL/NmT06NHUVf/+/T2PpgVOnDixefNm+16HCAsLczB430tSfn5+0dHR9r0mNBmd+fr16/I2c+bMPXv2vPWUW/j999+PHj1q2zlgwABSuHr1as6cOb/88kvb0QcOeevcuXNkZKT9QLLky5evY8eO9r13wIcffmjflQgHPeCtt94aOXKkfe9N/vnPf1rbs2bNOnTokMfBJKhWrZptDxPUjh07iD6vv/56r169YmNjbSc8WIiPKZo8Pfnxxx/Pnj1r3+sE9+oBdNl33nln2LBhERERVatWpbuEhobiAX/6X4SJprt27bLvNblHDwgMDHzhhReoL8b/p59+eru73DVe5wG00QcffECMvHLlSvHixXfv3m0/41aSj6Bp1gPogZZ63p0HrFmzhopio2HDhlRXTEyM59G0wJYtW1avXm3f6xDUHipg33u33CcPgEuXLlkue3ceQPBgUOzbt4+lAt3GdvSBc+3atRo1ajBa7QeS5a49ALm070pEqnmAZ2PdnQcwhMktkle4cOFketGDgoh7J+KVJMWKFTt16pR9rxPcqwewZmI4yfaNGzcuX74sHsByircMwhw5cmTMmJGebZgBpmnTplmyZDl9+nTlypVvF+9vt9/G7bKEDP79739nbWeYKzzGOXu++OKLTJkyEUjY+d5772XLlu3rr79u1aoVb9EXzmF6ZWVADulA7GT1nCdPnlsT/h9e5wG+vr7BwcGyTWxjrp88eTINVKBAgQ4dOjD7582bt3Tp0ryeO3du69atzz///J49ez755BNqgGZaunQpTcag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>