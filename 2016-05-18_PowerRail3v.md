**Author: PM**

**Suggested Tags:** raspberry pi, power, regulation, noise

# Exploring the 3.3V Power Rail 

The Raspberry Pi's power network consists of three power rails: 5V, 3.3V and 1.8V. Since the model B+ was released in 2014, a switch-based [buck converter](http://en.wikipedia.org/wiki/Buck_converter) is used to step down from 5V to the lower voltage levels. My focus in this article will be the operating characteristics of the 3.3V power rail which is not discussed as extensively as the 5V rail.

I'll be referencing the Raspberry Pi B+, my favorite version of the Pi, as I believe it has the best combination of connectivity features and power efficiency.  The concepts demonstrated with the B+ also apply to higher versions of the Pi.

## Power to the Pi    

Let's start by acknowledging that the 5V rail gets most of the attention and for good reasons.  It's directly connected to the external power source which can range from a "wall wart" AC adapter to a solar-charged battery.  This diversity of input sources makes the 5V rail subject to a variety of issues and concerns. 

The 5V rail is also the heavy hitter that supplies power to USB peripherals as well as HDMI.  It's also commonly used to power external sensors and devices that tap into the 5v power pins on the GPIO.  And the 5V rail provides power to the on-board buck converter.

The buck converter ([Richtek RT8020](http://www.richtek.com/en/Products/Switching%20Regulators/DC_DC%20StepDown%20Convertor/RT8020)) supplies power to the 3.3V rail. The primary job of the 3.3V rail is to power the SoC processor and GPIO. The 3.3V rail also offers surplus power which can supply external sensors and devices. For clarity, I should emphasize the surplus power I'm referring to is supplied from the 3.3V power pins on the GPIO, not the larger set of I/O pins on the GPIO that are dedicated to digital signal processing, an entirely different topic.

The surplus power from the 3.3V rail is more limited than what the 5V rail can deliver. We can estimate 3.3V surplus power by taking the rating of the converter (1A, per spec) and subtracting the load required by essential power consumers like the SoC and any GPIO signal connections.  Of course, the surplus power available will be lower when the Pi is running processor-intensive workloads and multiple signal processing applications.  But in many cases there should be sufficient power for low-power sensors or devices which operate at or near 3.3V.

We can always check the [spec](http://www.richtek.com/assets/product_file/RT8020/DS8020-08.pdf) to learn more about the converter.  But there's no substitute for hands-on testing to observe circuit behavior and confirm our understanding of what's happening.  So let's perform a few tests and draw some results-based conclusions.  The Raspberry Pi B+ I'm using for testing is running the Raspbian OS and is idling in a headless configuration (no monitor or keyboard/mouse attached).     

## Line Regulation 

One of the key roles of the Pi's buck converter is to regulate the output voltage to a constant level of 3.3V.  This is a critical function for the Pi as the SoC processor needs stable power.  The design of the Pi leverages the micro USB connector for input power, making common cell phone chargers a convenient solution to supply power. But the input voltage level can vary based upon the quality and rating of the device used to supply it.  So [regulation](http://www.electronic-products-design.com/geek-area/electronics/power-supply/line-load-voltage-regulation) is needed.

In this first test, I'll power the Pi from a bench linear power supply and I will manually adjust the input voltage from 3.0V to 6V in steps of 200mV.  Using a Digital Multimeter (DMM), I'll measure the output voltage at the 3.3V power pin for each input level.  Measurements will be entered into Excel for charting.  Keep in mind that the buck converter is always under load from the SoC which is idling during testing.

As you see in Figure 1, the converter outputs a steady 3.3V across the range of input voltage starting at 3.6V.  Below this level there is not sufficient headroom for the buck to convert the input voltage to 3.3V.  In this case, the output voltage will drop in proportion to the input level.  

![Figure 1](http://raspberrypise.tumblr.com/images/PowerRail3V/Figure1-LoRes.jpg "Figure 1")

<a href="http://raspberrypise.tumblr.com/images/PowerRail3V/Figure1-HiRes.jpg" target="_blank">Figure 1 - Click HERE for a larger image in a new browser window</a>

Overall, the converter is doing its job very well.  Consequently, you can have confidence that if you supply your Pi from a battery or solar panel with some fade in voltage, you will have quite a bit of margin to the downside before the 3.3V rail is impacted.

## Load Regulation 

Another role of the converter is to maintain a constant level of 3.3V across a range of load.  In this test, I'll use an active load tester connected to the 3.3V power pin and draw current from 0mA to 800mA in steps of 50mA.  Using a DMM, I'll measure the voltage at the 3.3V power pin at each step.  

I defined the upper range of current for this test at 800mA because I know from previous measurement that the SoC is drawing approx 200mA while idling.  So I don't want to exceed a total load of 1000mA or 1A on the converter as that is the upper limit of its normal operation (per spec).

As shown in Figure 2, the converter is maintaining a steady output voltage at 3.3V across the range of load. For comparison, I'm also showing voltage measurements taken at the 5V pin.  Note that unlike the 3.3V rail, there is a voltage drop on the 5V rail as load increases.  This is expected as some 5V components are subject to changes in resistance and conductivity under load and there is no regulation to compensate for this.  The 5V rail contains circuit protection solutions but they only kick in only when power levels exceed rated limits. 

![Figure 2](http://raspberrypise.tumblr.com/images/PowerRail3V/Figure2-LoRes.jpg "Figure 2")

<a href="http://raspberrypise.tumblr.com/images/PowerRail3V/Figure2-HiRes.jpg" target="_blank">Figure 2 - Click HERE for larger image in a new browser window</a>

## Power Noise 

As the old adage "You are what you eat" applies to humans, clean power is healthy food for microprocessors.  A switch-based buck converter always generates some noise during normal operation.  This noise appears on an oscilloscope as a oscillating waveform. If you are already familiar with  voltage ripple and noise generated by AC/DC converters, the noise from a DC/DC buck converter appears distinctly different.

Noise can be troublesome.  At higher frequencies it can interfere with RF devices.  At lower frequencies it can cause hum or static in audio devices.  The Pi's buck converter is rated at 1.5MHz.  This is a good frequency because it's higher than audio and lower than RF.  So there shouldn't be interference for most applications.

Noise is generally well-tolerated when the Peak-to-Peak voltage waveform (Vpp) is under 1% of the total voltage level.  In this case, we want Vpp to be less than 1% of 3.3V, or below 33mV to be considered acceptable. One could argue that the noise level should be even lower than 33mV to ensure reliable operation of the SoC and GPIO, but let's proceed with this general guideline for now. 

For this measurement, I'll supply a steady 5V input to the Pi and measure the noise at the 3.3V power pin using a USB oscilloscope. An active load tester will apply a load of 300mA to the 3.3V pin.  This dummy load of 300mA taken together with the 200mA load from the SoC will apply a total load of 500mA on the converter, just at the midpoint of its 1A rating.  

Figure 3 is a screenshot of the oscilloscope display.  Don't be alarmed by the busy waveform as the vertical scale is magnified to +/- 1.25mV.  It shows the Average Vpp at just under 1mV.  This value is well below the 1% noise guideline or any lower threshold one might propose.  

![Figure 3](http://raspberrypise.tumblr.com/images/PowerRail3V/Figure3-LoRes.jpg "Figure 3")

<a href="http://raspberrypise.tumblr.com/images/PowerRail3V/Figure3-HiRes.jpg" target="_blank">Figure 3 - Click HERE for larger image in a new browser window</a>

While I'm not providing screenshots for all the other noise tests I performed, I will summarize the results for you: The noise level did not change significantly whether I varied the voltage on the input side of the converter or varied the load drawn from the output side.  We can conclude that the buck converter is well-behaved with regard to keeping the noise level very low.   

## Sensitivity Analysis vs. Stress Testing

The testing just performed falls into the category of sensitivity analysis where we test over the normal operating range plus some margin on both sides.  These outcomes confirm what most of us probably expected.  But it's reassuring to test and get a visual grasp of the results.  This reinforces our understanding of the Raspberry Pi's power management.

As a next step, sensitivity analysis can be extended to stress testing for the same areas of regulation and noise we looked at above.  Testing voltage and current at extreme values will trigger circuit protection or cause destructive faults with blue smoke! This may be an interesting follow-on study for a future article.

