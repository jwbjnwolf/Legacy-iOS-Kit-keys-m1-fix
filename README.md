# Legacy iOS Kit, lil fix for failing to get firmware keys, and failing at iBSS reconnect on Apple Silicon macs.

### Failing to get firmware keys issue:
If you take a look in the `resources/firmware/<device>` folder, the `<build>` folder containing the `index.html` is directly in there.<br>
If you take a look in my created `html/firmware/<device>` folder, I've doubled the `<device>` folder.<br>

Why?

The HTML request that the `restore.sh` script makes looking for the device's `index.html` is looking for the `<device>` in the URL request twice. As a result the script fails with `(failed to get FirmwareJson from Server)` because the `http.server` launched by the script returns a `404`.

If you take a look at [this commit](https://github.com/jwbjnwolf/Legacy-iOS-Kit-keys-m1-fix/commit/40521ffde132454c05c3e27a449a71696a6cd167), I've commented out in the `restore.sh` file where it launches `http.server` on port `8888` using the `resources` folder as root. We want to instead spin up our own `http.server` @ port `8888` whilst inside the `http` folder so that's used as root. In order to do that follow below steps:

1) Create `html` folder, with an enclosing `firmware` folder`.
2) Copy the `<device>` you're wanting to that folder from `resources/firmware`.
3) Inside that copied `<device>` folder, create a subfolder of the same name as that `<device>` folder.
4) Move the desired `<build>` folder in this duplicated `<device>` folder. Delete the rest for tidiness.
5) Ensure you make [the edit to the restore.sh](https://github.com/jwbjnwolf/Legacy-iOS-Kit-keys-m1-fix/commit/40521ffde132454c05c3e27a449a71696a6cd167) so it doesn't try spinning it a `http.server` when we're going to run one ourself.
6) In terminal, in a new tab, `cd` inside the `html` folder.
7) Now run the `http.server` like so:
   ```
   python3 -m http.server 8888
   ```
8) On the first terminal tab opened inside the script root folder, run `restore.sh`.
9) It will now sucessfully get the firmware keys and you can terminate the `http.server` afterwards.

---

### iBSS waiting for reconnect failure on M1/Apple Silicon Macs:
If you're using an M1/Apple Silicon Mac, when the script is at the `iBSS` stage where it's waiting for the device to reconnect:
1) Quickly unplug the usb c to a adapter you're using in your mac,
2) Plug the adapter back in, and it'll get found then (choosing to trust the device if asked).

The device never seems to get picked up again o the iBSS reconnect if you do not unplug/replug tha adapter back in. [Cudos to voidf1sh](https://github.com/LukeZGD/Legacy-iOS-Kit/issues/171#issuecomment-1386241969) for figuring that out that physically disconnecting and reconnecting fixes this.

---

P.s: This is just a little cleaned up repo to highlight these two things. The two devices I had to try this tool with is an iPhone 5S global downgrading that to iOS 10.3.3, and an iPad 4 global downgrading that to iOS 8.4.1. Hence they're the only two devices  I've included in this repo for example. Get everything from the main repo and use this as a follow along if you run into these two issues with the firmware keys and iBSS reconnect fails.