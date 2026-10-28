# HW3 — Submission Template

This is the only file you need to fill out and submit for HW3. Copy it,
fill in your answers directly below each question, and turn in the
completed copy. You don't need to touch or resubmit `HW3.md`.

For multiple-choice items, mark your choice by changing `[ ]` to `[x]`.
For the Feedback section, several items just ask you to enter a single
number, see the instructions there.

---

# Part A: Multiple Choice (5 pts each, 20 pts total)

**MC1.** Why does a polling loop generally draw more power than an interrupt-driven equivalent while waiting?

- [ ] A. Polling requires a different arithmetic instruction
- [x] B. The CPU continues fetching and executing check instructions even when nothing has happened, while an interrupt-driven CPU can remain idle
- [ ] C. Polling loops use a slower instruction encoding
- [ ] D. Interrupts are only available on newer processors

**MC2.** What does the RISC-V `wfi` instruction allow the processor to do?

- [ ] A. Check a condition faster
- [x] B. Wait for an interrupt rather than continuously executing a polling loop
- [ ] C. Disable all interrupts permanently
- [ ] D. Increase its clock speed while waiting

**MC3.** What hardware component is responsible for receiving interrupt requests and signaling the CPU that an event requires attention?

- [ ] A. The compiler
- [x] B. The interrupt controller
- [ ] C. The register file
- [ ] D. The cache hierarchy

**MC4.** Why might a real system deliberately choose polling despite its potential power cost?

- [ ] A. Polling is easier to type
- [x] B. Interrupt handling and waking from low-power states have costs, and some applications may value predictable or very fast response enough to justify polling
- [ ] C. Interrupts are not supported on RISC-V
- [ ] D. Polling always uses less power in practice

---

# Part B: Show What You Learned

### N1 — The corrected model (20 pts)

In 2–3 sentences, correct this statement: "Polling and interrupts accomplish the same thing, so it doesn't really matter which one you use."

> Your answer: Polling and interrupt-driven waiting get you the same result, but they treat the CPU very differently while it waits. With polling, the CPU keeps running instructions just to check whether the event has happened. However, with interrupts, the CPU can drop into a low-power state until something wakes it. That can save energy, though how much simply depends on the hardware and on the cost of handling each interrupt.
> 
>
>

### N2 — Put real numbers on it (15 pts)

- **Microcontroller:** Renesas R9A02G021, a 32-bit RISC-V microcontroller.
- **Active mode:** Normal mode at 48 MHz, with peripheral clocks disabled and CoreMark code executing from flash - 7.80 mA typical.
- **Sleep/low-power mode:** Sleep mode at 48 MHz, with peripheral clocks disabled - 1.80 mA typical.
- **Active current is approximately _4.33_ times the low-power current:** (7.80 / 1.80 = 4.33).
- **Source:** Official Renesas R9A02G021 datasheet, Table 2.10, “Operating and standby current (1),” printed page 20. The measurements are typical values under specified test conditions. https://www.renesas.com/en/document/dst/r9a02g021-datasheet

### N3 — Catch the AI (20 pts)

- **What the AI claimed:** The AI claimed that interrupt-driven waiting using RISC-V's wfi instruction can save power by allowing the CPU to enter a low-power state instead of repeatedly checking for an event.
- **Why you questioned it or wanted more precision:** I questioned whether or not wfi actually saves power on a specific microcontroller because the RISC-V specification allows it to behave like a NOP. Also, the Renesas datasheet compares normal and sleep modes, rather than directly comparing polling with interrupt-driven waiting.
- **What you did to check it:** I closely looked through the Renesas R9A02G021 datasheet for its published normal-mode and sleep-mode current measurements. I also investigated the chip-specific hardware manual's description of wfi and sleep mode to determine if the instruction can actually put this microcontroller into a low-power state.
- **What you concluded:** The Renesas documentation indicates that wfi, under the right configuration, can put this microcontroller into sleep mode, and the datasheet reports substantially lower typical current in sleep mode than in normal mode. However, these figures don't directly measure the energy savings of interrupt-driven waiting compared with polling. A controlled experiment comparing both approaches under the same conditions would be needed to actually quantify that difference.

### N4 — Your best follow-up (10 pts)

**Your follow-up question:**

> Doesn't the fact that the RISC-V specification allows wfi to behave like a NOP, and that the Renesas datasheet doesn't directly compare polling with interrupt-driven waiting, mean we need more evidence before concluding that wfi actually saves power on this specific microcontroller? What exact evidence would establish that connection?

**Why that question was useful, or how it changed/refined your understanding (1–2 sentences):**

> This question helped me distinguish between what the RISC-V specification actually permits, what the microcontroller's hardware manual documents, and what the current measurements actually prove. I learned that showing sleep mode uses less current is not the same as directly measuring the energy savings of interrupt-driven waiting compared with polling.

### N5 — Connect back (10 pts)

Check the **two** lecture concepts that most directly explain the engineering tradeoff you investigated, and explain each in one sentence.

- [x] Interrupt controller — explanation: The interrupt controller receives and manages interrupt requests so the CPU can respond to events without repeatedly polling for them. 
- [x] `wfi` / low-power wait states — explanation: The wfi instruction can let the CPU enter a lower power sleep state while waiting for an interrupt, therefore reducing energy consumption compared with if it continuously executed polling instructions.
- [ ] Trap vectors and handler dispatch — explanation:
- [ ] Wake-up latency — explanation:
- [ ] Privilege mode transitions on interrupt entry — explanation:

---

# Appendix — AI Conversation

Paste your full AI chat export below. No cleanup required.

>**Im working on a computer architecture homework about polling versus interrupt-driven waiting and power consumption. Can you explain why polling generally uses more power than waiting for an interrupt? Please explain it at an intro level and include how the RISC-V wfi instruction relates to this. I want to investigate the topic. The key idea is that polling keeps the CPU checking whether something has happened, while interrupt-driven waiting lets the CPU stop doing that until it needs to respond. That difference can reduce power consumption.** 

Polling vs. interrupt-driven waiting Imagine a computer waiting for a keyboard input. 

Circuit board. Technology background. Central Computer Processors CPU concept. Motherboard digital chip. Polling: repeatedly check 

The CPU runs a loop that repeatedly asks, “Has a key been pressed yet?” If no key has been pressed, it checks again, and again. 

The CPU continues executing instructions. 

Those repeated checks consume energy. 

Much of the work accomplishes nothing if the input hasn't arrived. 

ARM Cortex-M Series Architecture | Tutorials on Electronics | Next Electronics Interrupt-driven waiting: wait for a signal 

Instead of repeatedly checking, the CPU waits. When the keyboard controller signals that input has arrived, an interrupt can wake the CPU so it can handle the input. 

The CPU can enter a low-power state while waiting. 

The hardware signals when attention is needed. 

The CPU spends less energy checking for an event that hasn't happened. 

Why does polling use more power? A simplified relationship is: 

Energy consumed 

Power × Time Energy consumed=Power×Time Polling can consume more energy for three main reasons: 

Instruction execution: The CPU repeatedly fetches and executes instructions to check the device. 

Active hardware: The processor and parts of its circuitry may remain active rather than entering a deeper low-power state. 

Wasted work: If an event takes a long time to arrive, the CPU may perform thousands or millions of checks that find nothing new. 

With interrupt-driven waiting, the CPU may be able to reduce its power consumption while idle, then briefly use more power when it wakes up to handle the event. 

One important distinction: polling does not necessarily consume the same amount of power in every implementation, and interrupts have overhead too. The biggest advantage often occurs when the CPU spends a long time waiting for an infrequent event. 

How RISC-V's wfi instruction relates to this wfi stands for Wait For Interrupt. It is a RISC-V instruction that tells the processor it can stall execution while waiting for an interrupt that may need servicing. The hardware can use this opportunity to reduce power consumption.  

RISC-V Documentation +1 

Compare these two simplified examples: 

Polling loop 

while (!event_occurred) { // Keep checking the event} handle_event(); 

The CPU repeatedly executes the condition check until the event occurs. 

Interrupt-driven waiting 

wait: wfij waitHere, wfi gives the processor an opportunity to stall. If execution resumes without an event that needs handling, the loop can execute wfi again. In a real interrupt handler, the processor would handle the interrupt and then return to the appropriate code. 

The important difference is that wfi allows the hardware to stop continuously executing the polling loop. 

One subtlety: wfi does not guarantee that the processor actually enters a low-power state. The RISC-V specification permits an implementation to treat it as a no-operation instruction (NOP). So, the power savings depend on the specific processor implementation.  

RISC-V Documentation +1 

What to investigate for your homework Since you want to investigate rather than just memorize an answer, I'd start with these questions: 

CPU activity: What work does the processor perform during a polling loop when no event has occurred? 

Power states: What can the hardware do while a processor is stalled by wfi that it cannot do as easily during continuous polling? 

Waiting time: Why might interrupt-driven waiting save more energy when events are rare or take a long time to arrive? 

Implementation: Why might wfi fail to save power on a processor that implements it as a NOP? 

A good primary source is the official RISC-V Privileged Architecture specification. Look for the section titled “Wait for Interrupt.”  

RISC-V Documentation 

The central concept to keep in mind is: polling spends energy repeatedly checking for work; interrupt-driven waiting can let the processor spend less energy until work arrives. wfi gives RISC-V hardware a way to support that second approach, but the actual savings depend on the implementation. 

**Ok great, you mentioned that wfi does not guarantee that the processor enters a low-power state and that some implementations can treat it as a NOP. Can you explain how that affects the power consumption? Also, could you give me an example of a specific RISC-V microcontroller with published active-mode and sleep/ low power mode current measurements, and point me toward its official datasheet so that I can verify the numbers myself? The important distinction is that wfi is an instruction-level hint, not a guarantee of a particular electrical power state. A processor can execute it without actually reducing its power consumption.** 

What happens to power if wfi is a NOP? The official RISC-V specification explicitly permits an implementation to treat wfi as a no-operation (NOP).  

RISC-V Documentation 

Case A: wfi actually stalls the CPU Lower potential power consumption 

The CPU stops executing instructions, and the hardware may gate clocks or enter a low-power state. 

Case B: wfi behaves like a NOP Potentially little or no power savings 

Execution continues immediately, so the surrounding loop may keep consuming energy while checking for events. 

Why does this matter? Consider the following loop: 

wait: wfij waitIf wfi stalls the processor, it can wait efficiently. If wfi is a NOP, the processor repeatedly executes wfi and j wait, consuming energy just as it would in another busy-wait loop. 

The exact current depends on the chip's hardware implementation, clock configuration, peripherals, and power-management settings. Even when wfi does stall the CPU, it does not necessarily turn off the entire microcontroller. 

A real RISC-V microcontroller with published current measurements A useful example is the Renesas R9A02G021. Its official datasheet includes both operating-current and sleep-mode measurements, so you can compare the two directly.  

Renesas +1 

R9A02G021 - Ultra-low Power 48MHz MCU with Renesas RISC-V CPU Core | Renesas Renesas R9A02G021 

32-bit RISC-V microcontroller 

Up to 48 MHz 

Published current measurements at 3.3 V 

Includes normal, sleep, and software standby modes 

Source: official Renesas datasheet, revision 1.10, February 2024.  Renesas 

Published current measurements The following values come from Table 2.10, “Operating and standby current (1),” pages 20–21 of the datasheet. The figures are typical current, not maximum guaranteed limits, unless otherwise indicated.  

Renesas 

Operating condition 

Typical current 

Normal mode, 48 MHz, peripheral clocks disabled 

7.80 mA 

Sleep mode, 48 MHz, peripheral clocks disabled 

1.80 mA 

Normal mode, 1 MHz, peripheral clocks disabled 

0.35 mA 

Sleep mode, 1 MHz, peripheral clocks disabled 

0.20 mA 

Software standby, 25°C, SRAM retained 

0.30 µA 

The 48 MHz normal-mode figure uses CoreMark code executing from flash; the sleep-mode figures are measured under the corresponding sleep-mode conditions. The standby figure uses a different, much lower-power state and should not be treated as equivalent to ordinary sleep.  

Renesas 

At 48 MHz, the typical current drops from 7.80 mA to 1.80 mA when moving from the specified normal mode to sleep mode: 

7.80 − 1.80 7.80 × 100 ≈ 76.9 % 7.80 7.80−1.80 ​  ×100≈76.9% That's about a 77% reduction in current under those specified conditions. It illustrates how much power can potentially be saved by entering an actual sleep state. 

However, this is not a direct measurement of polling versus wfi. The datasheet compares operating modes, not a polling program against an interrupt-driven program, and it does not establish that executing wfi alone produces those sleep-mode numbers. The distinction matters for your homework. 

Where to verify the numbers Official Renesas R9A02G021 datasheet 

72 pages · Revision 1.10 · February 2024 

Open the official PDF 

Go to pages 20–22 of the PDF (printed pages 20–22). Find Tables 2.10 and 2.11 for operating, sleep, and software standby current. 

How to connect this to your investigation I would keep these three ideas separate when writing your answer: 

Polling vs. interrupts describes how software waits for an event. 

wfi tells the RISC-V implementation that the hart can be stalled while waiting. 

Sleep or standby mode describes an actual hardware power state with measurable current consumption. 

The most interesting question is whether the microcontroller's implementation of wfi actually allows it to enter a lower-power state. If it does, the current may drop substantially; if wfi behaves like a NOP, that instruction alone may provide no such reduction. The official RISC-V privileged architecture specification, section 2.1.3.3 explains the architectural behavior.  

RISC-V Documentation 

That gives you a solid investigation: compare the architectural guarantee with the actual microcontroller's documented sleep behavior, without assuming that the instruction and the power state are the same thing. 

**Ok great, I looked through that data sheet. It gives 7.80 mA for normal mode and 1.80 mA for sleep mode at 48 MHz. That seems like a pretty significant difference, but I want to make sure I'm interpreting it correctly. What are the limitations of using these measurements to explain why interrupt-driven waiting can save power compared with polling? What additional information would I need to even be able to make a fair comparison?** 

You're interpreting the numbers in a reasonable way: 7.80 mA versus 1.80 mA is a substantial difference. The key limitation is that these measurements compare two hardware operating modes, not two waiting strategies. They support the idea that lower-power operation can save energy, but they don't directly prove how much energy interrupt-driven waiting saves compared with polling. 

What the measurements actually tell you Using the figures you found for the Renesas R9A02G021: 

Measurement 

Current 

Normal mode at 48 MHz 

7.80 mA 

Sleep mode at 48 MHz 

1.80 mA 

Difference 

6.00 mA 

The sleep-mode current is about 77% lower than the normal-mode current under the datasheet's specified test conditions. 

But there are three important limitations: 

Different hardware states: Normal mode and sleep mode may differ in which parts of the processor and its clocking system are active. The difference isn't necessarily caused solely by executing instructions versus waiting for an interrupt. 

Not a direct software comparison: The measurements don't tell us the current consumed by a polling loop compared with a program that executes wfi and waits for an interrupt. 

The same clock frequency doesn't mean identical conditions: Even when both measurements specify 48 MHz, other details—such as active peripherals, memory activity, clock gating, and test code—can affect the results. 

So, you can use these measurements to illustrate the potential benefit of sleep modes, but you shouldn't claim that they establish a 77% power saving from interrupt-driven waiting. 

What additional information would make the comparison fair? Ideally, you would measure the same microcontroller running two programs that respond to the same events. 

A fair experiment Test A Polling 

The CPU repeatedly checks a device's event flag until the event arrives, then processes it. 

Test B Interrupt-driven waiting 

The CPU waits using wfi, wakes when the same event arrives, and processes it. 

Keep the hardware, event timing, and work performed after each event the same. Measure current over the same total time period. Here is the information you would need to document. 

What to control or measure 

Why it matters 

Supply voltage 

Current alone does not tell you the complete power consumption if voltage differs. 

Clock frequency 

A higher clock frequency can increase active power. 

Peripheral configuration 

Enabled timers, communication devices, and clocks can contribute to current in both tests. 

Event frequency 

Rare events may make sleeping more beneficial than frequent events. 

Time spent waiting 

The longer the CPU waits, the more opportunity there is to save energy. 

Interrupt and wake-up overhead 

Interrupt handling and resuming execution also consume energy. 

Actual wfi behavior 

You need to establish whether the processor really stalls or enters a low-power state. 

Measurement duration 

You should include both waiting and event processing, rather than measuring only the lowest-current portion. 

The official Renesas datasheet is useful here because it explicitly distinguishes normal-mode measurements with CoreMark code executing from flash from sleep-mode measurements with peripheral clocks disabled. It also specifies conditions such as supply voltage and peripheral clock configuration. Those details are essential when interpreting the numbers.  

Renesas +1 

Compare energy over the same time interval For your homework, this is an especially useful distinction: 

P 

V I P=VI E 

P 

t 

V I 

t E=Pt=VIt where  P P is power,  V V is voltage,  I I is current, and  t t is time. 

For example, if both measurements were made at 3.3 V, the two datasheet currents correspond to: 

Normal mode:  3.3 × 7.80 

25.74  mW 3.3×7.80=25.74 mW 

Sleep mode:  3.3 × 1.80 

5.94  mW 3.3×1.80=5.94 mW 

Those are power estimates for the specified operating conditions, not total energy for handling an event. To compare polling and interrupts, you would need to account for how long the processor spends in each state, how often it wakes, and how much energy it uses to process events. 

A polling program might use more power while waiting but respond immediately when it finds an event. An interrupt-driven program might use less power while waiting but incur some wake-up and interrupt-handling overhead. The fair comparison considers the total energy consumed to wait for and handle the same events. 

The biggest unanswered question: does wfi actually save power on this chip? You would need to consult the R9A02G021 hardware manual and its documentation for sleep-mode entry and wake-up behavior. Specifically, look for: 

Whether executing wfi enters the documented sleep state. 

Which clocks and peripherals continue operating during sleep. 

Which interrupts can wake the processor. 

Whether any configuration steps are required before sleep entry. 

The official R9A02G021 datasheet and documentation page is the best starting point. The datasheet establishes the current figures, while the hardware manual is the more relevant source for the processor's power-state behavior.  

Renesas +1 

Bottom line: The 7.80 mA and 1.80 mA figures demonstrate that the chip can consume substantially less current in sleep mode. To demonstrate that interrupt-driven waiting saves energy compared with polling, you need a controlled comparison of both software approaches on the same hardware, including the time spent waiting, the cost of handling events, and evidence that wfi actually allows the processor to enter a lower-power state. 

**I want to make sure that my final conclusion is actually accurate. Based on the RISC-V specification and the Renesas datasheet, what can I confidently conclude about polling, interrupts, and wfi and what would be an overstatement? Also, of my lecture concepts including: interrupt controller, wfi / low-power wait states, trap vectors and handler dispatch, wake-up latency, and privilege mode transitions, which two most directly explain the power tradeoff, and why?** 

What you can confidently conclude Based on the official RISC-V specification and the Renesas R9A02G021 datasheet, I'd separate your conclusions into three levels of certainty. 

Supported by the specification 

Polling repeatedly executes instructions to check whether an event has occurred. Interrupt-driven designs can avoid that repeated checking by allowing hardware to notify the CPU when an event needs attention. RISC-V's wfi instruction permits the implementation to stall a hart while waiting for an interrupt, but the instruction can legally behave as a NOP.  

RISC-V Documentation 

Supported by the Renesas measurements 

The R9A02G021 datasheet reports typical current of 7.80 mA in normal mode and 1.80 mA in sleep mode at 48 MHz, under the specified test conditions. This demonstrates that its documented sleep mode can consume substantially less current than its documented normal mode.  

Renesas 

Not established by those measurements alone 

You cannot conclude that interrupt-driven waiting always consumes less power than polling, that wfi necessarily enters sleep mode, or that interrupt-driven waiting reduces current by exactly 77%. Those claims would require measurements of the two waiting strategies under comparable conditions. 

Which two lecture concepts most directly explain the power tradeoff? I'd choose (1) wfi / low-power wait states and (2) the interrupt controller. 

wfi / low-power wait states — the direct source of potential savings 

This explains why interrupt-driven waiting can consume less energy. Instead of repeatedly executing polling instructions, the CPU can stall and potentially enter a lower-power state. The actual savings depend on the implementation and whether the hardware really enters that state. 

Interrupt controller — how the CPU knows when to respond 

The interrupt controller manages interrupt requests and their delivery to the processor. It allows an external event, such as a timer expiring or a device receiving data, to notify the CPU instead of requiring the CPU to continuously check the device itself. Together with wfi, this enables efficient waiting for events. 

These two concepts connect the whole argument: the interrupt controller provides the notification mechanism, and wfi provides the opportunity to wait without continuously executing instructions. The RISC-V specification defines the wfi behavior, while the Renesas documentation describes the chip's interrupt-controller and low-power features.  

RISC-V Documentation +1 

Where the other concepts fit Lecture concept 

Relevance to power consumption 

Wake-up latency 

Important secondary factor: waking and responding take time and energy, and latency affects responsiveness. 

Trap vectors and handler dispatch 

Explains how execution reaches the interrupt handler; this adds work but isn't the main source of savings during a long wait. 

Privilege mode transitions 

Explains changes in execution privilege when traps are handled. These can add overhead, but a privilege transition is not inherently a low-power operation. 

One nuance: the interrupt controller alone doesn't guarantee lower power. An interrupt-driven program that never enters a low-power state can still consume substantial energy. Likewise, a processor that implements wfi as a NOP may keep executing its idle loop. 

A conclusion you can use in your homework Here's a concise way to bring the specification, measurements, and lecture concepts together: 

Polling can consume more energy than interrupt-driven waiting because the CPU repeatedly executes instructions to check for an event. Interrupt-driven waiting can avoid this repeated work by allowing hardware to notify the CPU when an event occurs. In RISC-V, the wfi instruction allows the processor to stall while waiting for an interrupt, potentially enabling a lower-power state. However, the specification permits wfi to behave as a NOP, so power savings are implementation-dependent. The Renesas R9A02G021 datasheet reports typical current of 7.80 mA in normal mode and 1.80 mA in sleep mode at 48 MHz under specified conditions. These measurements demonstrate the potential benefit of sleep mode, but do not directly quantify the difference between polling and interrupt-driven waiting. The two lecture concepts most directly related to this tradeoff are wfi / low-power wait states and the interrupt controller. 

That conclusion is appropriately qualified: it distinguishes what the ISA guarantees, what the datasheet measures, and what would require a direct experiment. 

**Great, I want to challenge one of your earlier claims: you said that interrupt-driven waiting can save power because the CPU can enter a low power state using wfi. But the RISC-V specification allows wfi to behave like a NOP, and the Renesas datasheet's normal-mode versus sleep-mode measurements don't directly compare polling with interrupt-driven waiting. Doesn't that mean we would need more evidence before saying for sure that wfi actually saves power on this specific microcontroller? What exact evidence would establish that connection?** 

Yes — your challenge is correct. To establish the connection on this specific microcontroller, we need evidence linking execution of wfi to entry into an actual low-power state, rather than relying only on the general RISC-V specification or comparing two datasheet current figures. 

I checked the official Renesas hardware manual, and there is an important piece of evidence for the R9A02G021: its section 10.6.1, “Transitioning to Sleep Mode,” explicitly states that executing WFI while the SBYCR.SSBY bit is 0 causes the MCU to enter Sleep mode. The manual also says that the CPU stops operating in this state while its internal registers are retained.  

Renesas 

That is stronger evidence than the datasheet current figures alone: it documents the actual behavior of this particular chip. 

What exact evidence would establish the connection? There are three levels of evidence, from documenting the mechanism to measuring the actual energy savings. 

Chip-specific documentation that wfi enters Sleep mode. 

This is now established by the Renesas hardware manual, section 10.6.1, page 190. It says that WFI with SBYCR.SSBY = 0 enters Sleep mode, where the CPU stops operating. It also explains that an interrupt cancels Sleep mode and starts interrupt handling.  

Renesas 

Read the official Renesas hardware manual 

Current measurements specifically associated with Sleep mode. 

The manual's Table 37.10, page 844, provides typical current measurements for both normal and Sleep modes. At 48 MHz with peripheral clocks disabled, it reports 7.80 mA for normal mode and 1.80 mA for Sleep mode, at 3.3 V.  

Renesas 

Combined with the documented WFI behavior, this supports the conclusion that executing WFI under the appropriate configuration can put this MCU into a state whose documented current is substantially lower than normal operation. 

One caveat: the current table does not measure a particular WFI program's current trace, so it is not direct experimental proof of the precise current drawn by your program. 

A controlled experiment measuring both waiting strategies. 

This would establish the strongest claim: that interrupt-driven waiting actually saves energy compared with polling on this particular chip. 

Run a polling loop and an interrupt-driven program on the same MCU. 

Configure WFI and the interrupt source correctly, with identical clock, voltage, peripherals, and event timing. 

Measure current over the same interval, including waiting, wake-up, and event handling. 

Repeat the measurements and compare total energy, not just the lowest-current moment. 

Ideally, you would also capture a current trace showing the transition into Sleep mode after WFI and the rise in current when an interrupt wakes the processor. 

So what can you say in your homework? You can now make a stronger, chip-specific statement than before: 

The Renesas R9A02G021 hardware manual documents that executing WFI with the appropriate configuration enters Sleep mode, where the CPU stops operating until the mode is canceled, including by an interrupt. Its published current measurements show substantially lower typical current in Sleep mode than in normal mode under specified conditions. This supports the mechanism by which interrupt-driven waiting can save power on this microcontroller. However, a direct measurement comparing polling with interrupt-driven waiting would still be needed to quantify the actual energy savings for those two programs. 

That is the most defensible conclusion: the manual establishes the WFI-to-Sleep connection, the datasheet establishes the lower-current Sleep state, and a controlled experiment would establish the actual savings compared with polling. 

 

---

# Feedback on the Assignment (5 free points)

Your opportunity to tell us about this assignment. There are no right or wrong answers, every item is optional, and leaving any item blank is fine. All items are graded on completion only, never on the response given. For the numeric items below, entering 6 or leaving the answer blank both count as "Prefer not to answer."

### B1 — Before

Before this assignment, how well could you explain why a polling loop and an interrupt-driven wait can produce the same outcome but cost very different amounts of power?

1. I could not explain this topic at all; it's new to me and something I'm here to learn.
2. I could name some of the related terms, but not explain the concepts behind them.
3. I could explain some of the assignment's key concepts at a high level, but not in detail.
4. I had some previous knowledge of this topic and would have felt comfortable explaining most of the key concepts.
5. I had a working background in this topic and was confident I could effectively explain it coming in.
6. (or blank) = Prefer not to answer.

**Your answer (1–6):** _1_

### P1 — After

After this assignment, how well could you explain why a polling loop and an interrupt-driven wait can produce the same outcome but cost very different amounts of power?

1. I still don't feel I could explain this comfortably yet; I need more practice and exposure to this topic.
2. I could explain some of the assignment's key concepts at a high level, but not in detail.
3. I could explain the individual concepts, but not fully how they connect.
4. I am confident that I could explain this topic well enough to help a fellow classmate understand it.
5. I could explain this topic confidently in a technical discussion with someone knowledgeable about the subject.
6. (or blank) = Prefer not to answer.

**Your answer (1–6):** _4_

### INV — What drove your investigation?

Pick the one that fits best.

- [ ] I mainly wanted to complete the required assignment efficiently.
- [ ] I mainly wanted to understand enough to answer the assignment questions correctly.
- [x] I mainly wanted to resolve something from the demonstration or assignment that didn't make sense to me.
- [ ] I mainly wanted to understand the technical issue well enough that I could explain or apply it beyond this assignment.
- [ ] Something else. *(Optional: tell us what.)*
- [ ] Prefer not to answer

### FAV — What helped most?

Pick one.

- [ ] The in-class demonstration / hook
- [ ] Reading the Concept section
- [x] The AI investigation
- [ ] Challenging or verifying something the AI told me
- [ ] Answering the Confirm questions
- [ ] Prefer not to answer

### SCA — What should we do with this format?

- [x] Continue as is. *Tell us more (optional): any suggestions to make it even better?*
- [ ] Adjust something. *Tell us more (optional): what would you change?*
- [ ] Stop, and go back to a traditional homework model. *Tell us more (optional): what didn't work for you?*
- [ ] Prefer not to answer

---

# Grading Rubric

| # | What's being evaluated | Points |
|---|---|---:|
| MC1–MC4 | Correctly identifies the four core concepts covered in this homework | 20 |
| N1 | States the corrected model: explains what the CPU is or isn't doing while waiting and why that matters for power | 20 |
| N2 | Provides a real, specific active-mode vs. sleep/low-power current comparison with a credible source | 15 |
| N3 | Identifies a specific AI claim, gives a genuine reason for checking it, describes a verification action, and reaches a supported conclusion | 20 |
| N4 | Identifies a specific follow-up question and explains how it advanced or refined understanding | 10 |
| N5 | Selects two defensible lecture concepts and correctly explains their connection to the investigated tradeoff | 10 |
| Transcript | Included and shows the AI investigation | Required |
| Feedback | Completes the required feedback items; "Prefer not to answer" (or leaving an item blank) counts as complete | 5 |

**Total: 100 points.**
