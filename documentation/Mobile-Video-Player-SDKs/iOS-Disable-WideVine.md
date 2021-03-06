---
layout: page
title: iOS Disable Widevine
subcat: iOS
weight: 150
---

[![iOS](https://img.shields.io/badge/iOS-Supported-green.svg)](https://github.com/kaltura/player-sdk-native-ios) 

## Description

Split our podspec to 2 subspecs - Core and Widevine.

`Core` - contains all source files, INCLUDING Widevine-related files.

`Widevine` - contains the Widevine library and a preprocessor macro WIDEVINE_ENABLED=1.

## How To Use
By defualt, the entire SDK is enabled.

### Entire SDK

To use the entire SDK, a client app has to use, as before:

```
pod 'KalturaPlayerSDK'
```

### Exclude Widevine

To exclude Widevine, only include the subspec Core:

```
pod 'KalturaPlayerSDK/Core'
```
