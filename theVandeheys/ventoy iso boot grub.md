

https://www.howtogeek.com/802328/how-to-boot-multiple-linux-distributions-with-ventoy/

Patrick Campanale / How-To Geek
If you’ve ever booted Windows or tested a Linux distro from USB, you know the routine: grab an ISO, run Rufus or Etcher, format, write, wait. A week later, want to try another distro? Back to square one. That cycle often made me put off exploring new distros.

Then I found Ventoy, and I haven't looked back.

The Problem With Bootable USBs
For a while, flashing USB drives worked fine for me, but once I started testing multiple Linux distros and experimenting more, that’s when the problems became clear.

Constant Reformatting: Each time I wanted to use a new ISO, I had to wipe the entire drive. Now, what if I wanted to go back to the older distro? Format the drive again.
One ISO per USB: The size of my flash drives didn’t matter (be it 128GB or 64GB). I was able to only use one ISO at a time, so the rest of the storage went unused.
Wasted Time: I’d get excited to try a new distro, but instead of getting to what I wanted, I had to wait for the ISO to copy. That wait often felt like an eternity, and sometimes the copy process would fail, forcing me to start over. Eventually, the joy of experimentation turned into frustration.
So, I resorted to buying multiple USB drives. This did solve a thing or two, but the process was still a mess. I had to remember the correct flash drive for each distro and then formatting the flash drive was still painful.

The Ventoy Solution
Diagram showing different partitions when creating USB drive using Ventoy
Ventoy
Balena Etcher is great, and so is Rufus. They both get the job done. But, my new favorite way of installing Linux is using Ventoy, which is a free and open-source utility for creating bootable USB. And the best part, you can store multiple ISO in a single flash drive, something I have been wanting for a long time. But how did I come across Ventoy?

Long story short, it wasn’t intentional; I just heard about Ventoy while browsing the internet and decided to check it out. At first, I didn’t realize you could copy multiple ISOs onto a single flash drive with Ventoy. This was a discovery I made over time when I thought, “If I can copy one ISO, why not more?” That’s when I discovered Ventoy’s true value, and I wished I had known about it sooner.

So, how does Ventoy even work? The idea of having multiple ISOs side-by-side on a single USB sounded confusing—or even impossible—since all I knew was the "one ISO per drive" rule.

Here’s what happens under the hood:

Instead of flashing an ISO, you install Ventoy onto your USB stick first. After that, you just copy the ISO files to it like any other file.
When you install Ventoy on a USB stick, it creates two partitions: a small boot partition that contains the Ventoy bootloader and a large data partition where you can freely copy your ISO files.
The bootloader (based on GRUB2) is what makes the magic possible. When your computer boots from the Ventoy USB, the bootloader scans the data partition for supported image files.
Once you pick an ISO from Ventoy’s boot menu, the tool dynamically maps that ISO into memory or presents it to the system as a virtual device. From the operating system’s perspective, it’s as if it were booted from a dedicated, properly flashed USB.
What Is the Experience Like?
Ventoy boot screen on a laptop.
Corbin Davenport / How-To Geek
Using Ventoy feels refreshingly simple. Imagine plugging in your USB, seeing a list of every ISO you’ve stored: Ubuntu, Windows, Fedora, or even rescue tools. Just press arrow keys and choose the ISO you want to boot from. No waiting, no re-flashing, no second-guessing. The best part? You can keep adding or deleting ISOs anytime. Just drag and drop them like normal files, and they’ll show up in the boot menu the next time you use the drive.

For me, the experience went from “Ugh, I need to re-flash my USB again” to “Let me just copy this ISO over and reboot.” It’s faster, cleaner, and feels like how bootable media should have always worked.

Why I Only Use Ventoy Now
After switching to Ventoy, I can’t imagine going back to the old way of flashing drives. For someone who enjoys distro hopping and experimenting, it makes testing new systems effortless. On top of that, Ventoy works across both BIOS and UEFI systems, so I don’t have to worry about compatibility issues. What really seals the deal is that it isn’t limited to Linux; it handles Windows and many other types of ISOs just as smoothly.

Another reason that makes Ventoy my favorite tool is ease of use. Setting up Ventoy for booting multiple Linux distros is quite simple. You just download the installer from the official website, run the installer and that’s it, your USB is now Ventoy-powered. From here on, just drag and drop ISOs onto the USB like normal files. When you boot from the USB, Ventoy will show you a menu of all the ISOs you’ve added.

PNY-Elite-Type-C-USB-3.2-ra-op
PNY Elite Type-C USB 3.2 Flash Drive
Brand
PNY
Capacity
256GB
Effortlessly transfer data across Type-C devices such as smartphones, laptops and desktops without a Type-C adapter. Built for convenience and durability; the sleek, light-weight, cap-less design leaves other ports available for use!

$21 at Amazon
Repeatedly flashing USB drives had its place, but for me, it feels outdated now. Ventoy has completely changed the way I handle bootable media. Once you try it, you’ll probably wonder, just like I did, why you spent so much time flashing drives in the first place