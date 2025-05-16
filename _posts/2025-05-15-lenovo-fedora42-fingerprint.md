---
title: Enabling Fingerprint Authentication on Lenovo ThinkPad T14 (Fedora 42)
date: 2025-05-15 13:24:00 +0530
categories: [Linux, Fedora, ThinkPad]
tags: [fingerprint, authentication, fprintd, Lenovo, ThinkPad T14, Fedora 42]
---

This post details the steps to enable fingerprint authentication on a Lenovo ThinkPad T14 running Fedora 42. We will use `fprintd` for managing fingerprint data and `authselect` to integrate it with the system authentication.

## Prerequisites

- You have a Lenovo ThinkPad T14 with a fingerprint reader.
- You are running Fedora 42.
- You have `fprintd` and `authselect` installed.

```bash
# Check if fprintd is installed
rpm -q fprintd

# Install fprintd and its PAM module if not installed
sudo dnf install -y fprintd fprintd-pam

# Check if authselect is installed
rpm -q authselect

# Install authselect if not installed
sudo dnf install -y authselect
```

## Steps

### 1. Enroll Fingerprints

First, we need to enroll your fingerprints. We'll start by enrolling the right index finger.

```bash
root@user1-thinkpadt14gen5:~# fprintd-enroll "user1"
Using device /net/reactivated/Fprint/Device/0
Enrolling right-index-finger finger.
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-retry-scan
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-completed
root@user1-thinkpadt14gen5:~#
````

The output shows the enrollment process for the right index finger. Follow the on-screen prompts to place your finger on the sensor when asked.

Let's also enroll the left index finger.

```bash
root@user1-thinkpadt14gen5:~# fprintd-enroll -f left-index-finger "user1"
Using device /net/reactivated/Fprint/Device/0
Enrolling left-index-finger finger.
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-finger-not-centered
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-finger-not-centered
Enroll result: enroll-stage-passed
Enroll result: enroll-stage-passed
Enroll result: enroll-completed
root@user1-thinkpadt14gen5:~#
```

Here, we explicitly specified `-f left-index-finger` to enroll the left index finger. You might see messages like `enroll-finger-not-centered`; try adjusting the placement of your finger.

### 2\. Verify Enrolled Fingerprints

You can verify if the fingerprints were enrolled correctly.

```bash
root@user1-thinkpadt14gen5:~# fprintd-verify "user1"
Using device /net/reactivated/Fprint/Device/0
Listing enrolled fingers:
 - #0: right-index-finger
Verify started!
Verifying: right-index-finger
Verify result: verify-match (done)
root@user1-thinkpadt14gen5:~#
```

This verifies the right index finger. Let's verify the left index finger as well.

```bash
root@user1-thinkpadt14gen5:~# fprintd-verify -f left-index-finger "user1"
Using device /net/reactivated/Fprint/Device/0
Listing enrolled fingers:
 - #0: left-index-finger
 - #1: right-index-finger
Verify started!
Verifying: left-index-finger
Verify result: verify-match (done)
root@user1-thinkpadt14gen5:~#
```

### 3\. Check `fprintd` Service Status

Let's check the status of the `fprintd` service.

```bash
root@user1-thinkpadt14gen5:~# systemctl status fprintd.service
○ fprintd.service - Fingerprint Authentication Daemon
    Loaded: loaded (/usr/lib/systemd/system/fprintd.service; static)
  Drop-In: /usr/lib/systemd/system/service.d
           └─10-timeout-abort.conf, 50-keep-warm.conf
    Active: inactive (dead)
      Docs: man:fprintd(1)

Apr 30 09:28:55 user1-thinkpadt14gen5.office.xyz systemd[1]: Starting fprintd.service - Fingerprint Authentication Daemon...
Apr 30 09:28:55 user1-thinkpadt14gen5.office.xyz systemd[1]: Started fprintd.service - Fingerprint Authentication Daemon.
Apr 30 09:29:39 user1-thinkpadt14gen5.office.xyz systemd[1]: fprintd.service: Deactivated successfully.
Apr 30 09:41:07 user1-thinkpadt14gen5.office.xyz systemd[1]: Starting fprintd.service - Fingerprint Authentication Daemon...
Apr 30 09:41:07 user1-thinkpadt14gen5.office.xyz systemd[1]: Started fprintd.service - Fingerprint Authentication Daemon.
Apr 30 09:42:24 user1-thinkpadt14gen5.office.xyz systemd[1]: fprintd.service: Deactivated successfully.
root@user1-thinkpadt14gen5:~#
```

The service appears to be starting and then deactivating. This is normal as `fprintd` is often activated on demand.

### 4\. Enable Fingerprint Authentication via `authselect`

Now, we'll use `authselect` to enable fingerprint authentication for system login and `sudo`.

```bash
root@user1-thinkpadt14gen5:~# authselect enable-feature with-fingerprint
Make sure that SSSD service is configured and enabled. See SSSD documentation for more information.

- with-fingerprint is selected, make sure fprintd service is configured and enabled

root@user1-thinkpadt14gen5:~# authselect apply-changes
Changes were successfully applied.
root@user1-thinkpadt14gen5:~# authselect current
Profile ID: sssd
Enabled features:
- with-mdns4
- with-fingerprint
root@user1-thinkpadt14gen5:~#
```

We enabled the `with-fingerprint` feature and applied the changes. The current profile now includes `with-fingerprint`.

### 5\. Testing Fingerprint Authentication

After enabling the feature, you should be prompted for your fingerprint in scenarios where password authentication is typically used, such as with `sudo`.

```bash
user1@user1-thinkpadt14gen5:~$ sudo -i
Place your right index finger on the fingerprint reader
9:45
sudo fwupdmgr update
9:46
```

As seen above, when `sudo -i` was executed, the system prompted for the fingerprint.

### 6\. Verify Fingerprint Sensor

You can also verify that the fingerprint sensor is recognized by the system using `fwupdmgr`.

```bash
user1@user1-thinkpadt14gen5:~$ sudo fwupdmgr get-devices |grep -i finger -A10
├─Fingerprint Sensor:
│    Device ID:          3715866xxx7778333qwqwd444sssieeexxxyzzzz
│    Summary:            Match-On-Chip fingerprint sensor
│    Current version:    01000366
│    Vendor:             Goodix (USB:0x27C6)
│    Install Duration:   10 seconds
│    Serial Number:      UIDSERIALNO_XXXX_MOC_B0
│    GUID:               1232rwew-8667-54c3-98e1-0988cxx7yy6z ← USB\VID_27C6&PID_6594
│    Device Flags:       • Updatable
│                         • Device can recover flash failures
│                         • Signed Payload
│                         • Can tag for emulation
```

This output confirms that a Goodix Match-On-Chip fingerprint sensor is detected.

### 7\. Listing Enrolled Fingerprints

You can list the enrolled fingerprints for a specific user.

```bash
root@user1-thinkpadt14gen5:~# fprintd-list "user1"
found 1 devices
Device at /net/reactivated/Fprint/Device/0
Using device /net/reactivated/Fprint/Device/0
Fingerprints for user user1 on Goodix MOC Fingerprint Sensor (press):
 - #0: left-index-finger
 - #1: right-index-finger
root@user1-thinkpadt14gen5:~#
```

This confirms that both the left and right index fingers are enrolled for `user1`.

## Conclusion

By following these steps, you should now have fingerprint authentication enabled on your Lenovo ThinkPad T14 running Fedora 42. You should be able to use your enrolled fingerprints for login and other authentication prompts.

-----
