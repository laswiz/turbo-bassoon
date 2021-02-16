<title>Chiller Control</title>
### [HOME](https://laswiz.github.io/turbo-bassoon/index.html) [About Me](https://laswiz.github.io/turbo-bassoon/AboutMe.html) [Enhanced Chiller Module](https://laswiz.github.io/turbo-bassoon/ChillerControl.html) [File Handling](https://laswiz.github.io/turbo-bassoon/FileHandling.html) [Databases](https://laswiz.github.io/turbo-bassoon/Databases.html)

### A Little Background...
My dad worked on a myriad of cooling equipment over his 40+ years of work. From small enviromental growth chambers to ice systems with 2,000,000 gallon tanks, he has seen it all. Being the curious kid, I was usually with him for all the times he was not in the office on a 'normal' day. He never programmed software. He was strictly mechanical and cooling related, and carried several licenses for handling different refrigerants at various times. He has more knowledge than I ever will have in this field, but as much as I could absorb and remember, I did.

## On Your Mark...
Enter my first lead developer project; a brand new machine. We were tasked with taking known functionality and reducing costs and overhead. With a team of four PLC developers, we completely rewrote 15+ years of code for a new machine. Four more developers completely remade the HMI (Human Machine Interface). We did it in one year. One of those facets was a chiller control for cooling purposes. Until this point, there were exactly two signals going to the chiller: On/Off and Error/No Error. Pretty simplistic for the 21st century. To save even more cost, we integrated this chiller control into our PLC. This gave the added benefit of having more data at our control. The Electrical Engineer flew over from Germany and he and I developed the PLC code for the chiller, including a nice little simulator we could test everything on. I continued to develop this chiller library until we had our first prototypes, and then debugged it, and made adjustments when necessary.

## Get Set...
Things went well at first. We had an open dialog and I communicated every error I found, every fix I made, and every test I did. I was methodical. I was meticulous. Then the responses started getting a little farther apart. Then I stopped getting responses. I learned a regime change at that company resulted in the engineer I was working with leaving. Then I started having cooling capacity issues, and fluctuations in the quality of the chillers we were receiving. I pleaded with them. I sent them traces, charts, raw data and error history files. They were stone faced. I ran my own tests proving that the capacity of the chiller had dropped and was no longer meeting our requirements. I was called a liar, uneducated, and just plain ignorant. I actually built an external heat exchanger and proved to them the issue I had seen. My director of Engineering then informed me I was no longer able to develop on the chiller control; that I would be using a precompiled library from my vendor. The hundreds of hours I spent tuning and making this better, the pages of data I gave them and support I gave them vanished. I reluctantly implemented this library. It lasted exactly 15 minutes. Then a fatal error, and a reboot of the control. A new library, wouldn't even start, just had errors. I was so upset that when the director of engineering asked me what had happened, I told him, "you did" and walked away.

## Go!
Once the political smoke had settled, my boss asked me quietly to fix it. A contact I had at the company sent me the source code in PDF format. I implemented it in a new library, but created an interface one of my own. This one I would control. If I couldn't change the chiller code, I would instead change the data going to the chiller! This opened up many different new functions we could implement. I could now make the chiller ready when the operator wanted to, protect it from freezing over a cold weekend, anticipate a heat load and adjust the cooling curves to compensate to remove problems with the design flaws. Everything I had wanted to do, I could now do with this external interface. So I did. And it worked. It worked well.... REALLY well. The chiller issues disappeared. Customers were happy again. The one nagging thing left was that I couldn't account for the different variability in the chillers themselves. Each one has a unique cooling curve. This curve can affect when the compressors shut down, and how much residual cooling remains (thus how low the water temperature could drift after the compressor shut down). So, for my capstone, one artifact I made was to take the time to create this function. I give it to you below. Just be sure to understand there is no way to test this. For one, I no longer work at that company. For two, that variety of machine is no longer produced. A newer, more efficient one with my base project code is made today, but the original one is no more. Interested? Drop me a line... As you can see, I can talk for HOURS on this subject.

	/* Cory Z 21-Jan-2021 *****************************************************
                                                                          
  	Name          :   chillerDynamicCoolingCalculation()                                        
                                                                           
  	Description   :   runs the dynamic cooling calculation for the chiller                                                          
                                                                           
  	Parameter     :   none.                                                  
                                                                           
  	Return Values :   none.                                               
                                                                           
  	Comments      :   This is only for the COAX laser, not the DISK
                                                                           
	****************************************************************************/

	void chillerDynamicCoolingCalculation (void) 
	{
	
		//this function is run from a state machine 
		switch (chiller.coolingCalc.state)
		{
			
			case IDLE:
				//idle state. Wait for the command to be set
				if ( chiller.commandInterface.calculateCoolingCurve == true ) 
				{
					chiller.coolingCalc.state = CALC_START;
				}
				break;
			
			case CALC_START:
				/************************************************************************************************/
				/*										         	*/
			        /*	The calculation works like this: 						 	*/
				/*		Run the chiller normal, and fire the laser at 100%			        */
				/*		Run normal until the temperature is below the setpoint, then turn off the laser */
				/*		Record the minimum temperature the chiller reaches 				*/
				/*		This is the residual cooling value						*/
				/*		Using this value, adjust the hysteresis to balance the cooling cycle 	        */
				/*											        */
				/************************************************************************************************/		
				laser.coolingCycleCalc.laserPower = 1000; //fixed decimal... 1000 = 100%
				laser.coolingCycleCalc.laserFrequency = 10000; //10kHz
				laser.coolingCycleCalc.beamOn = true; //fire beam
				chiller.coolingCalc.state = CALC_WAIT_FOR_COMPRESSORS;
				break;
			
			case CALC_WAIT_FOR_COMPRESSORS:
				//we wait here until both compressors are active 
				if ( chiller.outputs.comp1On == true && chiller.outputs.comp2On == true )
				{
					chiller.coolingCalc.state = CALC_WAIT_FOR_COMPRESSOR_OFF;
					laser.coolingCycleCalc.beamOn = false; //turn off beam
				}
				else if ( laser.coolingCycleCalc.reset == true )
				{
					memset(&laser.coolingCycleCalc, 0, sizeof(laser.coolingCycleCalc));
					chiller.coolingCalc.state = IDLE;
					chiller.commandInterface.calculateCoolingCurve = false;
				}
				break;
			
			case CALC_WAIT_FOR_COMPRESSOR_OFF:
				//start recording the minimum temperature
				laser.coolingCycleCalc.minCuTemp = laser.Inputs.chillerInputs.copperOutletTemp <= laser.coolingCycleCalc.minCuTemp ? laser.Inputs.chillerInputs.copperOutletTemp : laser.coolingCycleCalc.minCuTemp;
				//if the compressors are off, move to the next state 
				if ( chiller.outputs.comp1On == false && chiller.outputs.comp2On == false )
				{
					chiller.coolingCalc.state = CALC_WAIT_FOR_MIN_TEMP;
					//record the shut off temp 
					laser.coolingCycleCalc.compressorOffTemp = laser.Inputs.chillerInputs.copperOutletTemp;
				}
				else if ( laser.coolingCycleCalc.reset == true )
				{
					memset(&laser.coolingCycleCalc, 0, sizeof(laser.coolingCycleCalc));
					chiller.coolingCalc.state = IDLE;
					chiller.commandInterface.calculateCoolingCurve = false;
				}
				break;
			
			case CALC_WAIT_FOR_MIN_TEMP:
				//here we wait until the temperature has risen by 0.2° from our minimum. Then we know we have hit the bottom
				if ( laser.Inputs.chillerInputs.copperOutletTemp - laser.coolingCycleCalc.minCuTemp >= 2 )
				{
					//move to the calculation step
					chiller.coolingCalc.state = CALC_CREATE_HYSTERESIS;
				}
				else if ( laser.coolingCycleCalc.reset == true )
				{
					memset(&laser.coolingCycleCalc, 0, sizeof(laser.coolingCycleCalc));
					chiller.coolingCalc.state = IDLE;
					chiller.commandInterface.calculateCoolingCurve = false;
				}			
				break;
			
			case CALC_CREATE_HYSTERESIS:
				//knowing the residual cooling, we can offset the hysteresis so that under a full load we will not drop below the threshold for the second compressor to turn off
				//for display, write to a value 
				chiller.coolingCalc.residualCooling = laser.coolingCycleCalc.compressorOffTemp - laser.coolingCycleCalc.minCuTemp;
				//in order to never hit the value where the chiller shuts off both compressors, we double the residual cooling value and use this as our new delta 
				//Comp1 ON = setpoint + delta 
				//Comp2 ON = setpoint + delta + hysteresis
				//Comp1 OFF = setpoint + delta - hysteresis
				//Comp2 OFF = setpoint - delta - hysteresis
				laser.machineData.Chiller.delta = chiller.coolingCalc.residualCooling * 2;
				//we do need to adjust the hysteresis value to ensure we never shut off more than 1° below setpoint
				if ( laser.machineData.Chiller.delta + laser.machineData.Chiller.hysteresis > 10 )
				{
					laser.machineData.Chiller.hysteresis = 10 - laser.machineData.Chiller.delta;
				}
				//reset cycle 
				chiller.commandInterface.calculateCoolingCurve = false;
				memset(&laser.coolingCycleCalc, 0, sizeof(laser.coolingCycleCalc));
				hiller.coolingCalc.state = IDLE;
				break;
		}
	
	}

## Bonus
You made it this far, thank you! I wanted to add one little tidbit I haven't mentioned until now. One that really brings together my experience and that of my dad. We made 8 iterations of this chiller over the years. Different sizes, different options, all worked the same. One such version I was debugging and kept having issues when the second compressor would start. The low pressure would drop so fast that the expansion valve would snap closed, issue an error, and the machine would shut down. I tried everything I could think of. It was a design error (again), and I was starting to think there was nothing I could do about it other than shutting down the compressor when I saw this condition (which is bad when the chiller is under max heat because that two minutes the compressor cannot run would spell disaster for the laser). I called my dad and we talked about it. We must have talked for an hour. Finally he asked me if we had a hot gas bypass system in it. I said we did, but they wanted to remove it because it was inefficient and expensive (hot gas bypass reduces the cooling capacity but allows a finer control of the cooling). He told me to use it when ever the low pressure started to drop. So I did. And it solved the issue beautifully. In fact, it allowed me to more accurately hold the water temperature. Instead of +/- 1°C, I could hold 0.2°C! Eventually, I had to give it up, and they fixed the issue by having one expansion valve always open, but we proved it was possible, and without my dad, there is no way I could have done that.

