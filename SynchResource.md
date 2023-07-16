
Sync this resource

```

library ieee;
use ieee.std_logic_1164.all;


entity synchronizer is port (

			clk					: in std_logic;
			reset					: in std_logic;
			din					: in std_logic; -- pedestrian request input (assigned to pb(0) for ew and pb(1) for ns)
			dout					: out std_logic -- 
  );
 end synchronizer;
 
 
architecture circuit of synchronizer is

	Signal sreg				: std_logic_vector(1 downto 0);

BEGIN

process (clk, reset) is
begin
	-- 2 stage shift register
	-- using shift register, the input will be shifted twice before it synchronizes with the rest of the circuit
	
	-- the case where the reset button is pressed
	if (reset = '1') then 
		sreg <= "00";
		
	
	-- the case where reset button has not been pressed
	else 
	
		-- takes action only at the rising edge of the global clock (acting like DFF)
		-- shift the current rightmost bit one bit left such that right most bit is empty
		-- then replace the rightmost bit with the din
		
		if (rising_edge(clk)) then
			sreg <= sreg(0) & din;
		end if;
		
	end if;
	
	-- due to the shift, the leftmost bit should have our previous rightmost bit
	-- now, we can output the leftmost bit given that everything has been synchronized
	dout <= sreg(1); 
	
		
end process;


end;

```
