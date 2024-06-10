# Warthog - using the mining calculator to estimate your profitability
This guide will help you to estimate your mining profitability, depending on your mining hardware.

## Links :
- [Mining calculator](https://wartscan.io/calculator)
- [Hardware community database](https://docs.google.com/spreadsheets/d/1cQqlGm0wrrnlcPfjf0Cbsp-WEmtVAqVK_btHhbnhWao/edit#gid=1356787120)
- [Form to add your hardware to the database](https://docs.google.com/forms/d/e/1FAIpQLSfJBG2uoTuhF4fWvjFwXdp1uQrUQmijRjNS9KSOBBkXlVZXOA/viewform)
- [Janus scores sheet](https://docs.google.com/spreadsheets/d/11rSSp5sw82eOy6Rlm4H0eOYsLlP48E8mLZN0dA7Ry8s/edit#gid=0)

# How to use the tools ?

**the janusscore depends on your CPUs, your GPUs, and how they are associated on workers.**
So, you need to calculate the profitability of your worker separately. One worker = one motherboard and its CPU(s) and GPU(s).

- Knowing the hardware of your worker, you can calculate its janusscore depending of the CPU and GPU hardware.
- If you have several workers, calculate all janusscores separately, then add all of them to have your total janusscore.
- Your total janusscore will determine your profitability.

# Detailled procedure :

Search your worker CPU(s) and GPU(s) in our [hardware community database](https://docs.google.com/spreadsheets/d/1cQqlGm0wrrnlcPfjf0Cbsp-WEmtVAqVK_btHhbnhWao/edit#gid=1356787120) using Ctrl + F.
If you have several CPU and GPU, add all of their hashrate separately

Once you have your total CPU and GPU hashrate, write them **with the same unit** on the [mining calculator](https://wartscan.io/calculator), on the janusscore calculator. You will obtain your worker janusscore.
For example, if you have 26 MH CPU hashrate and 1.5 GH (=1500 MH) GPU hashrate :
![janusscore-calculator](/img/calculator002.png)

If you have several workers, repeat the operation to all of them to get your total janusscore (calculating all of them separately and adding them)
Now you have your total janusscore, convert it in GH (1 GH = 1000 MH) then use it on the calculator to knowyour estimated profitability in WART and USD.

Example with 578 MH janusscore = O.578 GH :
![janusscore-calculator](/img/calculator001.png)

