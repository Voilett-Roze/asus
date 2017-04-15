### About Asuswrt:

Asuswrt is a unified firmware developed by Asus for use in their recent routers.  The firmware was originally based on Tomato-RT/Tomato-USB, but has changed a lot since the original code fork.  Asus started using this firmware with the RT-N66U, but they also migrated some of their previous models to this new firmware, like the DSL-N55 or the RT-N56.

Being mostly based on GPL code, almost all the source code and all necessary build tools is available from their website.  There are a few proprietary components that are closed source (like the wireless drivers from Broadcom/Ralink).  In these cases, Asus includes binary-only versions of these files.  In the end, their GPL release includes everything needed to completely recompile a working firmware, with the exact same features as found in their firmware releases.

One big advantage of Asus going with a unified firmware is that they have one global code base for all their supported routers (with a few specialized branches as needed by some specific models).  That way, general bug fixes and improvements can automatically be applied to all other supported devices.  Similarly, new features added for one model can often be enabled for other models as well (provided the hardware can support it.)  And finally, it makes it easier for Asus to support older models for a longer time than if they were all running their own unique firmware.

### About Asuswrt-merlin

Asuswrt-merlin takes advantage of the fact that complete source code is available from Asus by building on top of it.  Asuswrt-merlin is my personal project, originally done for the RT-N66U, and eventually extended to support other flagship routers from Asus.

The general goal of this project is to provide an alternative to the original firmware, and remain in-sync with Asus's own development, so new features and bugfixes development by them can trickle down into Asuswrt-Merlin.  This means there are some very strict design guidelines behind this project:

1. **Stay as close as possible to the original firmware.**  By limiting the amount of sweeping changes made to the code, it means that whenever Asus releases a new version of Asuswrt, it generally takes only a few hours of work to merge their latest changes into Asuswrt-merlin (a bit more if it's a major new release from Asus).

2. **The goal is to improve, not to replace the original firmware's functionality**  Projects like DD-WRT and Tomato have existed for years, and benefit from those years of development to offer a lot of new features.  There is no point in reinventing the wheel - people looking for a completely different firmware with tons of advanced features should look at those excellent and well-established projects.

3. **Priorities: Stability > performance > features.**  Fewer changes to the code means fewer chances that new bugs might be introduced.  A router firmware is the core of your home network.  It must be rock-stable above all.  And performance optimizations can have unexpected side-effect when dealing with things that aren't fully understood.

4. **Targeting the novice and average user.** Asuswrt is designed to target both the novice and the average users.  This project will aim at the same target userbase.  There is enough doors left open with Entware and user-scripts so advanced users can fulfill their own requirements themselves.  Not overcrowding the webui with esoteric features will ensure that novice users won't be scared away.


Thanks to Entware/Optware and user-scripts, many features not integrated in the firmware can be manually added.  If you are advanced enough to need a feature, you will quite often be skilled enough to manually add it, or follow someone else's instructions on how to implement what you are looking for.
