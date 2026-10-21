# Petasos Settings overhaul: spec for sign-off

Nothing here is built yet. Mark anything you want changed, and I'll build once you approve.

## 1. How Settings works

- **Opening:** hold the mic button, as now.
- **Tabs:** the four preset buttons are the tabs.
  - **First set:** Device, Display, Bluetooth, Network.
  - **Second set:** Music, Soft keys, and two empty buttons.
- **Switching sets:** a short press of the far-right button swaps between the two sets, the same way it swaps the soft keys on the main screens.
- **Leaving:** a long press of the far-right button leaves Settings, as now.
- **Section labels:** the "— Versions —" header style is used for every section label in every tab.
- **Greyed-out rows:** they stay visible so you can see they exist, but the knob skips over them.
- **Checkboxes:** a checkbox row toggles when you press the knob.
- **Choice rows:** press the knob, turn it to change the value, and press again to save, as the Screen tab works now. The back button cancels the change.
- **Saving:** every change saves as soon as it's made. No tab has a separate Save step, except that Music keeps **Save and check** for its accounts.

## 2. Device

| Row | Values | Notes |
|---|---|---|
| Name | keyboard | Same as now. |
| Wake word | checkbox | "Hey Athena". |
| **— Versions —** | | |
| System image | read-only | |
| Petasos UI | press to check for updates | Renamed from "Petasos". |
| Hermes plugin | read-only | Renamed from "Plugin on Hermes". |

Three rows move out of Device:
- **Microphone** goes to each Bluetooth speaker.
- **VPN tunnel home** goes to each Bluetooth phone.
- **Soft keys** get their own tab.

## 3. Display

| Row | Values | Notes |
|---|---|---|
| Brightness | Auto, 10–100 % | Same as now. |
| Dim screen | checkbox | |
| Dim after | 15 / 30 / 45 / 60 s | Greyed out unless Dim screen is checked. |
| Dimmed brightness | 10–70 % | Greyed out unless Dim screen is checked. Turning the knob previews the level live, as now. |
| Turn off after | Off / 15 / 30 / 45 / 60 s | Moved from the old Screen tab. |
| **— Audio visualization —** | | |
| Bands | 1 / 3 / 5 | 3 is the new default, and 4 is dropped. Anyone on 4 moves to 3. |
| Response | Fast / Medium / Slow | How quickly the bands rise. Medium matches today's normal rise. |
| Decay | Fast / Medium / Slow | How quickly the bands fall. Fast, Medium and Slow match today's Quick, Gentle and Slow. |
| Peaks | checkbox | A peak-hold line on each band, thinner than the volume mark. It holds at the band's recent high for about a second, then drifts down. |
| **— Screensaver —** | | |
| Screensaver | Off / Clock / Now playing / Visualizer / Emblem / Automatic | Default is Off. |
| Start after | 60 / 90 / 120 s | Greyed out when the screensaver is Off. |
| Preview | press | Shows the chosen screensaver at once. Any button or a knob turn ends it and returns to this row. Greyed out when the screensaver is Off. |

**Moving from today's Fall-off setting:**
- Peaks becomes Response Fast with Decay Slow.
- Quick becomes Fast with Fast.
- Gentle becomes Medium with Medium.
- Slow becomes Medium with Slow.

**Screensavers.** Each one moves slowly so nothing burns into the screen.
- **Clock:** large time and date in the ΠΕΤΑΣΟΣ serif lettering, drifting.
- **Now playing:** album art, title and artist, with the progress along the bottom meander. When nothing is playing, it falls back to Clock.
- **Visualizer:** the sound bands at full screen, using the Audio visualization settings above. When nothing is playing, it falls back to Clock.
- **Emblem:** the Petasos emblem, fading slowly in and out.
- **Automatic:** Now playing while music plays, otherwise Clock.

**How the screensaver fits with the screen's other timers:**
- The screensaver starts after the idle time you choose. Dimming still follows its own timer, so the screensaver usually appears already dimmed.
- When "Turn off after" is set, the screen still turns off at that time, screensaver or not.
- The first press or turn after the screensaver appears only wakes the screen. It doesn't act on whatever was underneath.
- "Hey Athena" and an incoming answer from Athena both end the screensaver.

## 4. Bluetooth

This tab stays as it is now, with additions on each device's own page.

**A phone or computer** (anything that gives the Car Thing internet) gets a new section:

| Row | Values | Notes |
|---|---|---|
| **— VPN —** | | |
| Enabled | checkbox | |
| Configuration | one of the saved VPN configs | Greyed out unless Enabled is checked. |

While the Car Thing is online through that phone, it uses that phone's VPN setting. When you switch to another phone, the tunnel switches with it.

**A car or speaker** gets one new row next to the existing timing rows:

| Row | Values | Notes |
|---|---|---|
| Microphone | Internal (Petasos) / External (Bluetooth device) | Which mic is used while this device is connected. |

**Moving from today's settings:**
- Today's wireguard.conf becomes a saved config named "Home".
- Every paired phone gets VPN Enabled with "Home" if the tunnel switch is on today.
- If "The car's" microphone is chosen today, every paired speaker gets External.

## 5. Network

| Row | Values | Notes |
|---|---|---|
| **— Server —** | | |
| Address | keyboard | IP or hostname, e.g. 10.0.0.20:9119. Greyed out while signed in. |
| Username | keyboard | Greyed out while signed in. |
| Password | keyboard, shown as dots | Greyed out while signed in. |
| Status | see below | |
| **— VPN —** | | |
| VPN configurations | opens the list | |

**The Status row:**
- **Signed out:** "Logged out, press knob to log in". It's greyed out until Address, Username and Password are all filled in. Logging in uses the same sign-in the Mac app uses.
- **Signed in:** "Logged into Athenaeum as kayla, hold knob for 3 s to log out". It shows the server's name when Hermes reports one, otherwise the address. While you hold the knob, a progress bar fills along the row, and letting go early cancels.
- **Bad sign-in:** the reason ("Wrong password", "Can't reach 10.0.0.20") shows as the row's note.

Logging out stops everything that needs Hermes until you log in again. Music already in memory keeps playing, and Apple Music keeps working.

**The VPN configurations screen:**
- While it's open, the Network tab's label reads **VPN** and the other three tab labels go blank. The back button returns to Network and restores them.
- It lists every saved config by name, with the endpoint as its detail and a mark on any config a phone is using.
- "Add new VPN configuration" sits at the bottom of the list.
- Pressing a config opens its page with these rows:
  - **Name**
  - **Address**: the Car Thing's address inside the tunnel.
  - **DNS**
  - **Private key**: shown only as "Set". Pressing it offers **Generate a new key** or **Type a key**.
  - **Public key**: read-only, worked out from the private key. You give it to the server.
  - **Server public key**
  - **Endpoint**: host:port.
  - **Allowed IPs**
  - **Keepalive**
  - **Delete**: asks you to confirm, and is refused while a phone uses the config.
- "Add new VPN configuration" opens the same page empty, with a new private key already generated. That way the only long thing to type on the Car Thing is the server's public key.
- A config is checked when you save it, the same way wireguard.conf is checked today. A bad field is named in the note.

**Where configs live.** Each device keeps its own copy, because the VPN has to work when Hermes can't be reached.
- Configs added on the Hermes dashboard are copied to each device the next time it reaches Hermes.
- A config added or edited on a device is sent up to Hermes the next time it can reach it. This can't happen while the device is logged out.
- If the same config was changed in both places, the latest change wins.
- Private keys are stored on Hermes with owner-only access and are never shown again after saving, anywhere.

## 6. Music (second set)

The tab stays as it is now, with one new row under Streaming:

| Row | Values | Notes |
|---|---|---|
| Quality | as now | Saves at once (0.24.10). |
| Buffer before playing | 4 s / 10 s / 30 s / Whole song | Default 4 s. This applies to SoundCloud and Navidrome; Spotify plays on the phone. |

**How buffering works:**
- The next song in the queue starts downloading as soon as the current one has finished downloading, so a skip or track change doesn't wait.
- All buffering stays in RAM, and nothing is written to the flash. Each song is capped at about 64 MB. A longer song starts once 64 MB are in and streams the rest. A skip throws away the buffer for the song that was skipped.

## 7. Soft keys (second set)

This is the soft-key section moved as it is from Device:
- The first set and the "Second set, after the top-right button" set.
- Each key's page (Does, Asks, Label).
- "Use the shared soft keys".

It still needs Hermes, and it shows the same "Can't reach Athena" note when Hermes is out of reach.

## 8. Mac, Windows and Pi apps

The settings page uses the same tabs and order, minus what a computer can't do:

| Tab | Contains |
|---|---|
| Device | Name, Wake word, Microphone (the computer's mic choice, as now), Versions. |
| Display | Audio visualization (Bands, Response, Decay, Peaks) and Screensaver (Screensaver, Start after, Preview). There is no Brightness, Dim or Turn off, since the computer handles its own screen. |
| Network | Server (address, username, password, log in or out), VPN (Enabled, Configuration), and the list of VPN configurations with add, edit and delete. A config can be pasted in as a whole wg-quick file or imported from a .conf file. |
| Music | Same as the Car Thing. |
| Soft keys | Same as the Car Thing. |

- There is no Bluetooth tab.
- Logging out from a computer has a "Log out" button with a confirmation, not a 3-second hold.
- On a computer, the VPN setting belongs to the whole computer, not to a phone, so it moves from the phone pages to Network.

## 9. Hermes dashboard

- The Petasos page gets a **VPN configurations** section. It lists the configs with name and endpoint, and has **Add** (paste a wg-quick file or fill in the fields), **Edit** and **Delete**.
- Configs saved there reach every device as described in section 5.

## 10. How I'd ship it

1. **Release 1:** the Settings layout, Device, Display (without the screensaver), Bluetooth's Microphone row, Soft keys, Music's buffer setting and next-song loading.
2. **Release 2:** Network with sign-in and multiple VPN configs, per-phone VPN, the on-device VPN editor and the dashboard section.
3. **Release 3:** the screensavers.
4. **Release 4:** the Mac, Windows and Pi apps brought in line.

Each release carries the moves from today's settings, so nothing you've set is lost. I'll send screenshots of each Car Thing screen before each release.
