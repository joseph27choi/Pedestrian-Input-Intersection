# Turing-Machine

```
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

Entity State_Machine IS Port
(
	clk_input 										: IN std_logic;							-- global clock input
	reset											: IN std_logic;							-- reset input
	sm_clken										: IN std_logic;							-- clock enable input
	ew_req, ns_req									: IN std_logic;							-- pedestrian request inputs 
	blink											: IN std_logic;							-- blink input
	gns, ans, rns									: OUT std_logic;						-- output to display the green, amber, and red lights in NS direction
	gew, aew, rew									: OUT std_logic;						-- output to display the green, amber, and red lights in EW direction
	ns_cross_display, ew_cross_display				: OUT std_logic; 						-- output that displays when pedestrians can safely cross
	ew_output, ns_output							: OUT std_logic;						-- output to display pedestrian requests 
	ew_clr, ns_clr									: OUT std_logic;						-- clearing requests output
	state_number									: OUT std_logic_vector(3 downto 0)		-- displays the current state of the state machine
);
END ENTITY;

Architecture SM of State_Machine is
 
TYPE STATE_NAMES IS (S0, S1, S2, S3, S4, S5, S6, S7, S8, S9, S10, S11, S12, S13, S14, S15);   	-- list all the STATE_NAMES values

SIGNAL current_state, next_state				:  STATE_NAMES;     							-- signals of type STATE_NAMES

BEGIN
 --------------------------------------------------------------------------------
 -- State Machine: ew
 --------------------------------------------------------------------------------

 -- REGISTER_LOGIC PROCESS
 -- advances the state machine to a new state at periodic intervals
Register_Section: PROCESS (clk_input)  -- this process updates with the global clock
BEGIN
	-- when it detects the rising edge of the clock and the clock enable is active
	IF((rising_edge(clk_input)) AND (sm_clken='1')) THEN
		IF (reset = '1') THEN
			current_state <= S0; -- if reset is active, the state machine resets back to the beginning
		ELSIF (reset = '0') THEN
			current_state <= next_state; -- if reset is not active, advance to the next state
		END IF;
	END IF;
END PROCESS;	

-- TRANSITION LOGIC PROCESS

Transition_Section: PROCESS (ew_req, ns_req, current_state) 

BEGIN
	CASE current_state IS
		
		-- state0 and state1
		-- ew direction request monitoring period 
		-- if request is detected, jump to state6, else shift to normal next state
		WHEN S0 =>												
			IF((ew_req = '1') AND (ns_req = '0')) THEN			
				next_state <= S6;
			ELSE
				next_state <= S1;								
			END IF;
      	WHEN S1 =>		
			IF((ew_req = '1') AND (ns_req = '0')) THEN
				next_state <= S6;
			ELSE
				next_state <= S2;
			END IF;

		-- state2 to state7
		-- nothing special shifts to normal next state
      	WHEN S2 =>		
			next_state <= S3;	
      	WHEN S3 =>		
			next_state <= S4;
      	WHEN S4 =>		
			next_state <= S5;
		WHEN S5 =>		
			next_state <= S6;
		WHEN S6 =>		
			next_state <= S7;
		WHEN S7 =>		
			next_state <= S8;

		-- state8 and state9
		-- ns direction request monitoring period
		-- if request is detected, jump to state14, else shift to normal next state
		WHEN S8 =>		
			IF((ew_req = '0') AND (ns_req = '1')) THEN
				next_state <= S14;
			ELSE
				next_state <= S9;
			END IF;
		WHEN S9 =>		
			IF((ew_req = '0') AND (ns_req = '1')) THEN
				next_state <= S14;
			ELSE
				next_state <= S10;
			END IF;

		-- state10 to state15
		-- nothing special, shifts to normal next state
		WHEN S10 =>		
			next_state <= S11;
		WHEN S11 =>		
			next_state <= S12;
		WHEN S12 =>		
			next_state <= S13;
		WHEN S13 =>		
			next_state <= S14;
		WHEN S14 =>		
			next_state <= S15;
		WHEN S15 =>		
			next_state <= S0;
		WHEN OTHERS =>
         next_state <= S0;
		
	END CASE;
END PROCESS;
 
-- DECODER SECTION PROCESS 
-- sets the State Machine output signals when the state machine reaches specific states
Decoder_Section: PROCESS (current_state) 

BEGIN
	CASE current_state IS

		-- State 0
		WHEN S0 =>		
			-- NS: flashing green light on
			gns <= blink;
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "0000"; -- state number: 0
			
			-- checks if there is a pedestrian request in the EW direction and none in the NS direction
			-- displays request on LEDs 
			IF ((ew_req = '1') AND (ns_req = '0')) THEN 
				ew_output <= ew_req;
			ELSE 
				ew_output <= '0';
			END IF;
			
		-- State 1
		WHEN S1 =>
			-- NS: flashing green light on
		 	gns <= blink;
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "0001"; -- state number: 1
		
			-- checks if there is a pedestrian request in the EW direction and none in the NS direction
			-- displays request on LEDs 
			IF ((ew_req = '1') AND (ns_req = '0')) THEN 
				ew_output <= ew_req;
			ELSE 
				ew_output <= '0';
			END IF;
			
		-- State 2
		WHEN S2 =>	
			-- NS: green light on
		 	gns <= '1';
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '1'; -- pedestrian in NS direction can cross
			state_number <= "0010";	 -- state number: 2
			
		-- State 3
		WHEN S3 =>	
			-- NS: green light on
		 	gns <= '1';
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '1'; -- pedestrian in NS direction can cross
			state_number <= "0011";  -- state number: 3

		-- State 4
		WHEN S4 =>	
			-- NS: green light on
		 	gns <= '1';
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '1'; -- pedestrian in NS direction can cross
			state_number <= "0100";	 -- state number: 4

		-- State 5
		WHEN S5 =>
			-- NS: green light on
		 	gns <= '1';
			ans <= '0';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '1'; -- pedestrian in NS direction can cross
			state_number <= "0101";  -- state number: 5
				
		-- State 6
		WHEN S6 =>
			-- NS: amber light on
		 	gns <= '0';
			ans <= '1';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross 
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "0110";	 -- state number: 6	
			
			-- if there is a pedestrian request in the NS direction, it is fully implemented at State 6
			-- clears the request becuase the request is already implemented 
			ns_output <= '0'; 
			IF (ns_req = '1') THEN
				ns_clr <= '1'; -- request is cleared
			ELSE 
				ns_clr <= '0';
			END IF;
				
		-- State 7
		WHEN S7 =>	
			-- NS: amber light on
		 	gns <= '0';
			ans <= '1';
			rns <= '0';

			-- EW: red light on
			gew <= '0';
			aew <= '0';
			rew <= '1';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "0111";	 -- state number: 7

		-- State 8
		WHEN S8 =>
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: flashing green light on
			gew <= blink;
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1000";	 -- state number: 8	
			
			-- checks if there is a pedestrian request in the NS direction and none in the EW direction
			-- displays request on LEDs 
			IF ((ew_req = '0') AND (ns_req = '1')) THEN 
				ew_output <= ew_req;
			ELSE 
				ew_output <= '0';
			END IF; 

		-- State 9
		WHEN S9 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: flashing green light on
			gew <= blink;
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1001";	 -- state number: 9
			
			-- checks if there is a pedestrian request in the NS direction and none in the EW direction
			-- displays request on LEDs 
			IF ((ew_req = '0') AND (ns_req = '1')) THEN 
				ns_output <= ns_req;
			ELSE 
				ns_output <= '0';
			END IF; 
		
		-- State 10
		WHEN S10 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: green light on
			gew <= '1';
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '1'; -- pedestrian in EW direction can cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1010";	 -- state number: 10
		
		-- State 11
		WHEN S11 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: green light on
			gew <= '1';
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '1'; -- pedestrian in EW direction can cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1011";	 -- state number: 11
		
		-- State 12
		WHEN S12 =>
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: green light on
			gew <= '1';
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '1'; -- pedestrian in EW direction can cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1100";	 -- state number: 12

		-- State 13
		WHEN S13 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: green light on
			gew <= '1';
			aew <= '0';
			rew <= '0';

			ew_cross_display <= '1'; -- pedestrian in EW direction can cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1101";	 -- state number: 13

		-- State 14
		WHEN S14 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: amber light on
			gew <= '0';
			aew <= '1';
			rew <= '0';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1110";	 -- state number: 14

			-- if there is a pedestrian request in the EW direction, it is fully implemented at State 14
			-- clears the request becuase the request is already implemented 
			ew_output <= '0'; -- LED turn off 
			IF (ew_req = '1') THEN 
				ew_clr <= '1'; -- request is cleared 
			ELSE 
				ew_clr <= '0';
			END IF;

		-- State 15
		WHEN S15 =>	
			-- NS: red light on
			gns <= '0';
			ans <= '0';
			rns <= '1';

			-- EW: amber light on
			gew <= '0';
			aew <= '1';
			rew <= '0';

			ew_cross_display <= '0'; -- pedestrian in EW direction cannot cross
			ns_cross_display <= '0'; -- pedestrian in NS direction cannot cross
			state_number <= "1111";	 -- state number: 15

	END CASE;
END PROCESS;

END ARCHITECTURE SM;
```
