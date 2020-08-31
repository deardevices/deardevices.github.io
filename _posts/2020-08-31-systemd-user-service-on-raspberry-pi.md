---
layout: post
title: "Lingering Services -- How to prevent systemd from killing my processes after I log off?"
image: "/assets/lazy_tiger.jpg"
---

This is about some recent experience I had with systemd on the Raspberry Pi. The idea was to set up a process to continuously run in the background as user 'pi', so I decided to give systemd's *user* services a try (as opposed to the *system* ones). It turns out I missed some important detail in the docs, which led to surprising results.

## My Setup: Two Raspberry Pis, one with a camera
There are two devices involved: *CapturePi* for capturing images from its camera, and *PullPi* to pull those images at a certain rate via ssh. While for *PullPi*'s task a simple cronjob is probably sufficient, I wanted *CapturePi* to have a full-blown service that would permanently run in the background.

Let's take a look at the simpler one first. Here's the relevant line of the crontab for user *pi* on *PullPi*:
```
*/10 6-21 * * * /home/pi/get_latest_image.sh
```

During the day, some shell script is called every ten minutes. This is not shown here, but its job is to connect to *CapturePi* via ssh and copy the latest camera snapshot to a local folder.

As a side note: You might be asking yourself **why we even want to have two devices** for this task. And if so, why not just trigger the camera capture from *PullPi* right away, instead of having two things going on asynchronously?

I like this approach for a few reasons: first of all, the internet connection of *CapturePi* is not very reliable. The device might just go offline for a few hours during the day and then re-appear on the radar. Also, the device doesn't provide a way for the user to look at the images it has captured -- it's behind a NAT router whose configuration we don't want to touch. Furthermore, I want *CapturePi* to sample images more often than just every ten minutes, and it should **store them locally, no matter how bad its network connection might be** (so I may download them later).

Now back to the main topic: For setting up the second part, the systemd service on *CapturePi*, I followed [a tutorial on github](https://github.com/torfsen/python-systemd-tutorial). I created a service for the user *pi* and enabled it so it would start automatically when the system boots up.

The configuration file for the service looks like this:
```
# ~/.config/systemd/user/capture.service

[Unit]
Description=My Capture Service

[Service]
ExecStart=/usr/bin/python3 /home/pi/capture_every_minute.py
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=default.target
```

This configures systemd to execute `capture_every_minute.py` for the user *pi*. In addition, the environment is manipulated to make the script's output show up in syslog properly. The script (not shown here) contains an endless loop to capture an image from the camera once every minute.

To enable this service (so it would start on system boot) we run:

```
systemctl --user enable capture
```

Note that none of these actions required root access to the device. This does feel a bit suspicious to me -- but we will come to that later.

After setting up and testing all of this, **everything seemed to work just fine**.

## When weird behavior began
The issues suddenly appeared when I started logging off of the *CapturePi* system, mainly because the debugging phase was over, I assumed (but actually it was about to begin just now).

I started to notice huge variations in the sampling interval of the system. While the images on *PullPi* were sampled every ten minutes, just as expected, the other ones (stored on *CapturePi* locally) had timestamps I found hard to reason about:

Some images were sampled correctly once every minute, whereas others had a timestamp difference of about ten minutes. This is surprising because one of the main goals was to do the faster sampling on *CapturePi*, regardless of the polling interval of *PullPi*.

From the logs (`/var/log/syslog`), it looks like systemd would kill the new service every ten minutes:
```
Aug 29 18:00:21 pi systemd[597]: capture.service: Main process exited,
    code=killed, status=15/TERM
Aug 29 18:10:08 pi systemd[597]: capture.service: Main process exited,
    code=killed, status=15/TERM
Aug 29 18:20:11 pi systemd[597]: capture.service: Main process exited,
    code=killed, status=15/TERM
```

It almost seems like systemd killing the process has something to do with sample intervals getting longer -- but how?

Let's put together what we know by now:
1. During initial testing, images were always sampled once per minute.
2. Later, the sampling rate dropped (but only for certain periods).
3. Systemd kills the capture process every 10 minutes.

The 10 minutes interval looks very suspicious. Is this some systemd default setting: re-start user processes every 10 minutes? Googling this behavior doesn't reveal anything.

## Checking the docs again...

Maybe reconsidering the systemd docs is a good idea. What's the difference between *system* and *user* services, anyway?

An [article on the Arch Linux wiki](https://wiki.archlinux.org/index.php/Systemd/User) provides a good hint about *user* services:
> This process will survive as long as there is some session for that user, and will be killed as soon as the last session for the user is closed.

Oh, this might explain why systemd suddenly started killing the process: During initial testing and debugging, there was always an ssh connection established -- keeping *one last session* open, and therefore preventing systemd from killing the service.

To be fair with the original tutorial I followed: it turns out this behavior is also mentioned there, and they even tell you how to fix it:
> You can make your user's systemd instance independent from your user's sessions [...] via `sudo loginctl enable-linger $USER`

**Wow, that's great:** enabling *User Lingering* prevents systemd from killing the service -- so that fixes the issue.

## But what about the 10 minutes?

Sure, it's working now. That's good. Nevertheless, there's still a part of the story I find hard to explain: at the times when no user was logged in, why did the service keep capturing images? Well, it happened at a lower rate. But why did the service run *at all* during those periods?

Remember that cron job pulling the latest image via scp? Triggered via a cron job?

Well, it turns out that this is enough to start a *session* for the user *pi* on *CapturePi* and make the capture script run just a single time. :-)

## Summary
- Systemd user services seem to be intended for processes to be run as long as a user is logged in.
- For services running longer, either use 'lingering' user services or system services.

Title image by <a href="https://pixabay.com/users/Pexels-2286921/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1285229">Pexels</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1285229">Pixabay</a>.
