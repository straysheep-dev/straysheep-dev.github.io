---
title: "Review: WiFiChallenge"
icon: material/wifi-cog
draft: false
#date:
#  created: 2025-10-16
#  updated: 2025-12-14
categories:
  - training
  - cwp
  - wireless
  - wifi
  - hacking
  - docker
---

# :material-wifi-cog: WiFiChallenge Exam & Course Review

!!! abstract "Overview"

    This is a consolidated reflection on the CWP exam and course, as well as the WiFi Challenge Lab open source project.

    In summary, this is an incredibly practical and effective wireless pentesting course.

    - [academy.wifichallenge.com](https://academy.wifichallenge.com/)
    - [Full lab and Docker files: github.com/r4ulcl/WiFiChallengeLab-docker](https://github.com/r4ulcl/WiFiChallengeLab-docker)
    - [Blogs and walkthroughs by the course author: r4ulcl.com/posts](https://r4ulcl.com/posts/)

<!-- more -->

**Strengths & Weaknesses Overview**

In comparing strengths and weaknesses, you'll notice there aren't a lot of real weaknesses.

The main weakness isn't really a problem with the course so much as the platform missing a global, precise, search function. This would help you find the exact page or relevant content you might be looking for, but without it you're encouraged to take better notes.

I'm leaving out anything here that could be considered a [feature request](#closing-thoughts-feature-requests) to grow the course itself.

| **Strengths** | **Weaknesses** |
| --- | --- |
| 🟢 Affordable | 🔴 Course platform search capabilities (2025) |
| 🟢 Effective and practical | ⚪ Docker lab ***may*** require tweaking in some setups |
| 🟢 Teaches pentesting ***and*** building + working with wireless | |
| 🟢 Current, updated, with wide tool coverage | |
| 🟢 Nzyme IDS integration | |
| 🟢 Self-contained course material | |
| 🟢 Offline, local, customizable lab | |
| 🟢 Comparison of physical WiFi adapters |
| 🟢 Exam feedback and help | |

---


## :octicons-code-review-16: Context & Review

When signing up for this course I had already completed the OSWP exam from OffSec. That was largely thanks to the dockerized pentesting lab *this* course provides you to practice in offline, and modify as needed.

I spent the majority of the year after purchasing the course getting lost with Docker, Ansible, Molecule, GitHub actions, and Packer, after seeing what this project achieved through emulating wireless networks using the [mac80211_hwsim kernel module](https://www.kernel.org/doc/html/latest/networking/mac80211_hwsim/mac80211_hwsim.html). This resulted in a prolonged tangent into devops, and the project to convert my entire tech stack into infrastructure-as-code. At a certain point I got the alert that I needed to schedule the exam, since it had almost been an entire year.


### :simple-bookstack: Course Material

The depth of the course is evident if you read through the source code of the public project, look at what it's doing, and think about what attacks are possible based on that. [I was even sidetracked by Nzyme](./nzyme-rpi-proxmox-tailscale.md), setting that up in my lab for real world use which has been one of the more interesting and useful side projects, since it takes IDS into the physical world.

The course walks through every network type, for ***creating*** (yes, creating), connecting to, and attacking, WiFi. That's the strength of this course, you'll know how everything works at a practical level. You can go further and set up your own RADIUS server and authentication through Active Directory if you want, but you never need to memorize how to do things like this because ***templates*** exist both in the course, and in some of the tools designed to attack those networks. Really useful if you want to build your own AP, for example.

As for introducing you to, and showcasing popular tools, the course somehow figured out exactly what top tools to focus on. All of those highlighted give you (require?) a certain level of control and configuration to use them. What that means is, everything is just hands on enough to teach you how it all works without getting too far into the weeds on doing everything manually, from scratch, which can feel very clunky with some attacks. By the end, you'll feel comfortable digging into the details of the tools discussed as well as those not covered. This should also give you confidence running some of the more automated tools, to know what they're doing and/or why they're failing if they do.


### :material-console-network: Virtual Study Environment

Both the labs and the exam were done within my [Kali packer build for wireless pentesting](https://github.com/straysheep-dev/packer-configs/tree/main/kali-linux) in QEMU / virt-manager, and building the Docker images locally with a few modifications.

Pre-built VMs are also available to download. Ultimately thanks to Docker you can do this on any compatible VM and in any hypervisor.

!!! tip ""

    My [troubleshooting notes on Docker are at the bottom of this post](#docker-usage-troubleshooting), because the prebuilt Docker images didn't always work perfectly for me.

I went this route to familiarize myself with setting up these tools and to see what isn't available in Kali by default. What can really help you, if you want to automate some of this into your own Kali builds, is reviewing [the repo's shell script that installs the attacker tools](https://github.com/r4ulcl/WiFiChallengeLab-docker/blob/main/Attacker/installTools.sh).

If you're more comfortable in Debian or Ubuntu, a lot of these tools support those distros just as much as they do Kali.


### :material-code-greater-than: Preparing for and Taking the Exam

!!! tip ""

    Everything related to the exam process (scheduling, starting, and running the exam) is painless, leaving you to focus on preparing for, and simply executing on, the exam.

The exam goals and time limit felt like they were balanced just right. When it came down to it, I had only a few weeks to really focus on this and pause all other personal projects. Realistically that translated to 3 weekends, and 1-4 hours on about half of the nights of any week. At that point I had no practial knowledge or muscle memory with wireless pentesting, with my only experience being the OSWP. I had an idea of what the attack paths were, so it wasn't difficult to recall previous, and absorb new concepts, quickly. My constant work on devops projects likely helped me catch up on researching, note taking, and thinking about "what do I need to move ahead right now". The answer to that question will be different for everyone, but that's your key to success here.

The point is, despite how in-depth the course material can be, it's very straight forward and easy to follow. I believe even those getting started with Linux will have a clear path to success here with persistence and curiosity alone.

!!! tip ""

    The course provides you a methodology overview, not as prescriptive as the OWASP WSTG but enough to guide you without a walkthrough.

I suggest trying to complete the entire Docker lab using just methodology notes from the course, to see where your gaps in understanding are. The [Cheat Sheets](#cheat-sheets) section below is where my knowledge gaps were resolved. Even though it's somewhat redundant information, reading other approaches to options in attack paths is what made me realize things like having one ongoing recon window running in the background for reference was useful, as well as what attacks are best to kick off and let run while you review and digest other information.

!!! tip ""

    The final key here goes back to [advice](https://www.youtube.com/watch?v=bJ4gJVXPAS0) that [BB King (BHIS)](https://www.blackhillsinfosec.com/team/brian-king/) has mentioned anytime report writing comes up; write the report as you go.

This may not be the most popular method but I fall into that category; I just cannot take a folder of disparate screenshots and txt files, and turn those into a coherant report after the fact. Adding to this advice, what I do is create a report skeleton for a wireless pentest in general. Something that could even work as a base in the real world. Then I wrote a couple of mock reports using the learned methodology moving through the entire Docker lab. This will highlight what you might run into with report writing, that you wouldn't want to figure out *during* the exam.


### Closing Thoughts & Feature Requests

!!! question "Final Thoughts"

    This course covers nearly every WiFi attack path using a number of the most up-to-date and popular tools, with room to add more outside of the foundations it sets if it really wanted to. The possibilities there are exciting, because if you've done any live recon or research on [wigle.net](https://wigle.net/), you'll know that WiFi is seemingly overlooked as an attack vector compared to everything else "within" the cloud or network that has a ton of tooling available, engineering time spent on it, and frankly, ease of visibility into it. It's a lot like firmware in that sense, we have sysmon, auditd, and endless security tooling on the network and within the OS, but we don't have the same insights into the UEFI world at runtime.

!!! note "Feature Requests"

    I shared some of these requests with the course author after completing the exam. Since the lab and Docker code is open source, I don't believe this spoils anything by mentioning it, but it's implied in the name of "WiFi" Challenge Lab that its focus is on pentesting WiFi.

    - Recon + attacks on other protocols (Bluetooth, LoRa, SubGHZ, RFID, Infrared, NFC, Z-Wave, Zigbee, etc)
    - More focus (maybe even a dedicated course + exam) on the "blue team" side using Nzyme
    - Red team considerations outside of pentesting


### :material-note-plus: Remaining Notes

Some of the details above were intentionally vague to prevent spoiling anything. The rest of my notes below though will be more specific, and were chosen to share here because they were the most helpful to me and can be applied generally even outside of the lab.

Part of the reason for this post is to solidify the entire lab experience in memory by reviewing it after clearing the exam, since I don't do wireless pentesting every day. I've also been using the WiFiChallenge lab repo since before the course existed, so in some sense these notes are the summary of everything I've learned there, so I don't forget them. This allows me to retrace my steps in the future.

Ultimately, my goal is using pentesting to discover and validate the defense + hardening measures to secure things. I can take that back to the devops world and attempt to bake that into my infrastructure using code.

---


## :simple-wikibooks: Cheat Sheets

Both the pwnbox and eaphammer wikis were referenced throughout the course, as both are very popular repos. A few of these sections specifically proved invaluable for me in tying all of the concepts together that I was still wondering about in the back of my mind after completing the majority of the course content at least once. That will vary for everyone, but these are highlighted here more for me to come back to more than anything.

!!! tip "This is all Methodology"

    Having a checklist is one thing, but when it becomes intuitive, the whole process feels less cumbersome and more like an investigation. This is where you want to be before the exam.

- [github.com/koutto/pi-pwnbox-rogueap/wiki](https://github.com/koutto/pi-pwnbox-rogueap/wiki)
    - [Mindmap of WiFi hacking workflow](https://github.com/koutto/pi-pwnbox-rogueap/blob/main/mindmap/WiFi-Hacking-MindMap-v1.png)
    - [Comparison of EAP Methods](https://github.com/koutto/pi-pwnbox-rogueap/wiki/07.-WPA-WPA2-Enterprise-(MGT)#comparison-of-eap-methods)
    - [MITM commands with Bettercap](https://github.com/koutto/pi-pwnbox-rogueap/wiki/MitM-Commands)
    - [When and how Rogue AP attacks are effective](https://github.com/koutto/pi-pwnbox-rogueap/wiki/08.-Evil-Twin-Attacks)
- [github.com/s0lst1c3/eaphammer/wiki](https://github.com/s0lst1c3/eaphammer/wiki)

---


## :octicons-devices-16: Operational Tips

These are things I eventually started doing, or realized along the way, that I found useful. Some of this is comes down to preference, while a few of the notes I just wasn't seeing the first few times I looked at something.

!!! tip ""

    - Dedicate an entire tmux pane (no splitting) to a long running triage + capture on all channels / networks, just to see what you might find over time
    - You can run multiple captures simultenously, like a general long-running triage capture and a surgical capture on one AP + channel
    - Get real-time visibility into your radios; I did this by creating [check-iw.sh](https://github.com/straysheep-dev/linux-configs/blob/main/check-iw.sh)
    - EAPhammer has the ability to perform karma, ***and*** captive portal attacks
        - Karma attack types are covered by both wikis above, in the [Cheat Sheets](#cheat-sheets) section
    - A note on decloaking hidden APs via bruteforce name guessing:
        - mdk4 actually runs very fast, and only prints out one of every 300 or so attempts to console
        - You may have realized [SecLists](https://github.com/danielmiessler/SecLists) and similar wordlists don't have a top-1000 WiFi SSIDs, but [wigle.net/stats#ssidstats](https://wigle.net/stats#ssidstats) does (as a downloadable CSV!)

---


## :material-checkbox-multiple-marked-circle: Methodology Highlights

This section details actions or steps I found most helpful to note when creating my workflow. The technical how-to or method is best demonstrated in the course, along with other public wikis and references.

!!! question ""

    - Start with the low-hanging fruit first
    - Similar to wired network recon automation, start long-running or broad capture(s) in the background as needed
    - The general rule of bruteforce operations still applies here:
        - Run these tasks asynchronously in the background if it's possible and safe
        - If it's not a prerequisite to an attack path, or the attack path itself, bruteforce should be your last resort
    - Focus on one target at a time, gather the relevant evidence and log it
    - Review and note the relationships of all targets in scope, these will define attack opportunities

---


## :simple-docker: Docker Usage & Troubleshooting

You can copy the [docker-compose-local.yml](https://github.com/r4ulcl/WiFiChallengeLab-docker/blob/main/docker-compose-local.yml) build file, modify it, and build the entire lab locally using the Dockerfiles:

!!! note "AI Usage"

    I shared what I was running into with GPT 5, for ideas and code snippets, so I could take that to the Docker documentation and review. That led to the `-d --build --force-recreate --remove-orphans` suggestion, since it's `docker compose up --help` and not `docker compose build --help` that unintuitively has those options. The block below is what I landed on after some conversation and research to help me consistently build, modify, and rebuild the lab environment as needed.

```bash
# Build & run (daemonized) using a specific file
docker compose -f docker-compose-local.yml up -d --build

# Check status
docker compose ps

# Restart the containers if you run into issues, optionally specify a name for only one
docker compose restart [name]

# Get a shell on the container to debug
docker exec -it WiFiChallengeLab-APs /bin/bash

# Completely rebuild the same containers, but with modified dockerfiles
docker compose -f docker-compose-minimal.yml up -d --build --force-recreate --remove-orphans
# From there you'll be able to continue to start / stop the containers without needing to rebuild them.
```

**Customized Local File**

This is my customized version of the "main" dockerfile, which runs only the AP's + clients. By building them all locally you can sometimes avoid issues if you're running into problems with the pre-built docker images.

!!! important "Point-in-Time"

    This may not be necessary in your case, or even work in the future if the project updates it's compose file. It's simply an example that worked at a point-in-time.

```yml
services:
  aps:
    #image: r4ulcl/wifichallengelab-aps:latest
    build: ./APs/ # uncomment to build the Docker file
    restart: on-failure   # Automatically restart on failure
    container_name: WiFiChallengeLab-APs
    env_file: ./APs/.env
    volumes:
      - ./certs:/root/certs/:ro
      - ./certs:/root/mgt/certs/:ro
      - ./certs:/var/www/html/.internalCA/
      - /lib/modules:/lib/modules
      - ./logsAP:/root/logs/
    healthcheck:
      test:
        - CMD-SHELL
        - ip netns exec ns-ap /bin/bash -c '
          curl -f -s http://localhost/login.php >/dev/null || exit 1;
          curl -s http://localhost:8080 >/dev/null || exit 2;
          if [ $(ps aux | grep host_aps_apd | grep -v grep | grep -c host_aps_apd) -ne 15 ]; then exit 3; fi'
      interval: 5s
      timeout: 5s
      retries: 3
      start_period: 30s
    network_mode: host #NETNS
    privileged: true #NETNS

  clients:
    #image: r4ulcl/wifichallengelab-clients:latest
    build: ./Clients/ # uncomment to build the Docker file
    restart: on-failure   # Automatically restart on failure
    container_name: WiFiChallengeLab-Clients
    env_file: ./Clients/.env
    volumes:
      - ./certs:/root/certs/:ro
      - /lib/modules:/lib/modules
      - ./logsClient:/root/logs/
    depends_on:
      - aps
    network_mode: host #NETNS
    privileged: true #NETNS
    healthcheck:
      test:
        - CMD-SHELL
        - ip netns exec ns-client /bin/bash -c '
          curl -s http://localhost >/dev/null || exit 1;
          if [ $(ps aux | grep wpa_wifichallenge_supplicant | grep -vE "grep|sudo|timeout" | grep -c wpa_wifichallenge_supplicant) -lt 17 ]; then exit 2; fi'
      interval: 5s
      timeout: 5s
      retries: 3
      start_period: 45s
```
