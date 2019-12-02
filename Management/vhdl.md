# VHDL 
A circuit is represented in VHDL as a **module**, which consists of two files:
- A `source`, specifying how the circuit works in terms of its `logical` inputs/outputs (e.g. a `button` can be mapped to a `led`)
- A `constraints` file, specifying how `logical` inputs/outputs are mapped to `physical` inputs/outputs on the board (e.g. `button` denotes the `A5` port on the device, which can be found by looking at the FPGA's datasheet)

# Source
The source portion is divided in two main parts:
- **Entity declaration**, which defines inputs and outputs port - and it is similar to a function declaration in C.
- **Architecture block**, defining the entity's behaviour (as in the function body in C)

```vhdl
entity example_code is port(
    port_1 : in std_logic;
    port_2 : out std_logic_vector(1 downto 0)
);

architecture example_code_arch of example_code is
<Code>
end example_code_arch;
```



