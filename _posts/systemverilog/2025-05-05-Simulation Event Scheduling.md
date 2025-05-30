---
number: 5
title: "SV5. Simulation Event Scheduling in Verilog/SystemVerilog"
date: 2025-05-05
categories:
  - SystemVerilog
tags:
  - event scheduling
  - delta cycles
  - non-blocking assignments
excerpt: "Deep dive into simulation event scheduling, delta cycles, and non-blocking assignments. Understand how Verilog simulators handle timing and event ordering in digital designs."
header:
  teaser: /assets/images/5- Simulation Event Scheduling.png
classes: wide
---



![Simulation Event Scheduling](/assets/images/5- Simulation Event Scheduling.png)

## Video Tutorial

Watch this comprehensive video explanation of simulation event scheduling:

<div class="video-container" style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 20px 0;">
  <iframe 
    src="https://www.youtube.com/embed/dLiQp9lyVKg" 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
    frameborder="0" 
    allowfullscreen>
  </iframe>
</div>

In Verilog or SystemVerilog, the simulation process is divided into Time Slots, and each Time Slot is divided into event regions. This structure is called the simulation event scheduler. 

Each event region includes Active, Inactive, and Non-Blocking Assignments (NBA) regions for design-related events and a Monitor region for system tasks.

The simulation can cycle through Active, Inactive, and Non-Blocking Assignments (NBA) regions multiple times in a Time Slot to respond to all scheduled events.

## 1. Active Region

- **Execution of Processes**: Procedural blocks (`initial` and `always`) are executed.
- **Evaluation of Continuous Assignments**: The right-hand side of continuous assignments (`assign`) is evaluated and then the left-hand side is updated.
- **Blocking Assignments**: The right-hand side of blocking assignments (`=`) is evaluated and then the left-hand side is updated.
- **Evaluation of Non-blocking Assignments (RHS)**: Only the right-hand side (RHS) of non-blocking assignments (`<=`) is evaluated.

## 2. Inactive Region

- **Zero Delay Blocking Assignments**: Blocking assignments with a zero delay (`#0`) are executed.

## 3. NBA (Non-Blocking Assignment Update) Region

- **Updating LHS of Non-blocking Assignments**: The left-hand side (LHS) of all non-blocking assignments is updated. This two-step evaluation and update process of NBA is for modeling the behavior of clock-to-Q characteristics of flip-flops.
- **Scheduling New Events**: Updates in the NBA region can initiate additional events, potentially looping back to the Active region for the same time slot. This process is called a Delta Cycle.

## 4. Monitor Region

- **System Tasks**: System tasks like `$monitor` and `$strobe` are executed. These tasks print the final values of signals at the end of the time slot.

## Simulation Example

```verilog
module event_scheduler_demo;
  reg a, b, c;
  
  initial begin
    a = 0; b = 0; c = 0;    // Initial values
    
    // Time 0
    a = 1;                  // Blocking - immediate update in Active region
    b <= a;                 // Non-blocking - RHS evaluation in Active, LHS update in NBA
    c = b;                  // Blocking - uses old value of b (0)
    
    // Delta cycle causes re-evaluation after NBA updates
    #0 $display("After first set: a=%b, b=%b, c=%b", a, b, c);
    
    // Time 10
    #10;
    a <= 0;                 // Non-blocking - evaluated at time 10, updated in NBA
    b = a;                  // Blocking - uses current value of a (1)
    c <= b;                 // Non-blocking - RHS evaluated (1), LHS updated in NBA
    
    #0 $display("After second set: a=%b, b=%b, c=%b", a, b, c);
  end
endmodule
```


