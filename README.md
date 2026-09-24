TwinOLED — Scrolling Text Marquee
A Two-Screen Wireless Ticker
What It Is
TwinOLED is a project that uses two ESP32 boards, each with its own small OLED display, to create one wide, seamless scrolling text ticker. The two screens sit side by side and act as a single 256-pixel-wide canvas, so text scrolls smoothly from one screen to the other as if they were one continuous display.

The two boards talk to each other wirelessly using ESP-NOW — a low-latency, peer-to-peer wireless link built into the ESP32. No router, no WiFi network, no internet. Just two chips talking directly.

The Core Idea
Picture two OLED screens sitting next to each other:

   ESP32 "A"        ESP32 "B"
   (left half)     (right half)
Together they form a virtual 256×64 canvas. A message starts off the right edge, scrolls left across Board B, crosses seamlessly onto Board A, and exits the left edge. Then it loops.

To the viewer, it looks like one wide display. Under the hood, it's two independent boards coordinating in real time.

How the Two Boards Work Together
Board	Role	Responsibility
ESP32-A	Simulator + Left Renderer	Owns the scroll position, draws characters in x = 0–127, broadcasts every frame
ESP32-B	Right Renderer	Listens for the broadcast, draws characters in x = 128–255
This simple split removes any ambiguity about who owns the scroll — Board A drives, Board B mirrors.

The MAC Address Problem — and the Easy Fix
Normally, ESP-NOW is like mailing a letter: to send data from one board to another, you must know the receiver's exact MAC address (a unique hardware ID). Every ESP32 has a different one, so you would have to look it up, write it down, and hardcode it into your program. That's a hassle — and it breaks the moment you swap boards.

The Solution: Broadcast
Instead of mailing a letter to one specific address, we shout into the room. In ESP-NOW, this is called broadcast mode. The sender doesn't need to know anyone's MAC address — it just sends the packet out to everyone listening on the same channel.

In code, this is a one-line change: replace the target MAC with the broadcast address — six bytes, each one the hex value F F.

You can think of it as writing "EVERYONE" on the envelope instead of one person's name. Any board in range hears the message.

The whole MAC lookup problem disappears. No discovery code. No hardcoding. No pairing step.

Two Small Details That Make It Clean
1. The echo problem. When a board shouts, it also hears its own packet come back. Instead of fighting that, we simply decide that only one board listens. Board A talks and never registers a receive callback; Board B listens and never sends. No conflict, no echo issue.

2. Strangers in the room. In a space with other wireless devices, you might hear packets that aren't yours. To filter them out, every packet carries a small tag byte — a known "magic" number. If the tag doesn't match, the packet is dropped. Simple, reliable, no errors.

Hardware
Per board (×2):

1× ESP32 development board (any variant with WiFi)

1× SSD1306 OLED display (128×64, I²C)

4× jumper wires (VCC, GND, SDA, SCL)

USB power source

No sensors. No buttons. No MPU. No extra modules.

Wiring
OLED	ESP32
VCC	3.3V
GND	GND
SDA	GPIO 21
SCL	GPIO 22
Identical on both boards.

Software Stack
ESP-NOW — built into the ESP32 Arduino core, no library to install

Adafruit SSD1306 — for the OLED

Adafruit GFX — for text rendering and automatic clipping

A small shared packet struct: the message text, the scroll position, and the magic tag

Why the Boundary Is Seamless
Both boards share the same virtual coordinate system. Board A covers x = 0 to 127, Board B covers x = 128 to 255. Each board draws the entire message at the virtual position and lets the OLED library automatically clip anything outside its own 128-pixel window.

Result:

Characters at x = 0–127 appear on Board A only

Characters at x = 128–255 appear on Board B only

Characters outside 0–255 appear on neither

No manual split logic. Each board just draws the whole string — the hardware quietly handles the rest.

The only rule is: Board B must subtract exactly 128 from the virtual x coordinate. Off by even one pixel and you get a visible stutter at the seam.

What You Can Do With It
Desk ticker — scroll a personal message, quote, or countdown

Shop window sign — display promotions or opening hours

Event display — show a greeting or schedule at a conference

Weather / news feed — pull data from WiFi and scroll headlines

Clock + message — combine a live clock with a scrolling tagline

Two-line ticker — split the height into two independent scrolling rows

Because the base is just "text on a shared canvas," the message can come from anywhere: hardcoded, typed over Bluetooth, fetched from a server, or triggered by a sensor.

What It Demonstrates
ESP-NOW low-latency peer-to-peer wireless

Broadcast mode — no MAC lookup, no pairing, no discovery code

Distributed rendering — one logical canvas split across two devices

Master/slave coordination — one board drives, one mirrors

Automatic clipping as a design tool — no manual boundary math

A reusable foundation for games, dashboards, and multi-board displays

Tuning Options
Change	Effect
Scroll speed	How fast the message moves
Frame delay	Smoothness of motion
Text size	Bigger or smaller characters
Vertical position	Where the text sits on screen
Message string	What actually scrolls
The One-Line Summary
TwinOLED — two OLED displays, one scrolling canvas, linked wirelessly by ESP-NOW, with no MAC lookup and no pairing — just a message that flows seamlessly from one screen to the next.

