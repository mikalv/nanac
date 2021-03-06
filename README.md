# Nanac

Nanac is a tiny Python two-pass assembler and a C virtual machine. The VM is under 150 lines of C and is fully modular allowing you to expand quickly, the machine uses on 32bit opcodes:

 * 8bit module ID
 * 8bit cmd ID
 * 8bit arg A
 * 8bit arg B

New operations are registered as modules containing commands in your programs source code,
the module and cmd IDs are printed to stdout for use by the assembler.

There are `2^8` registers, each is a native sized `void*` pointer.

There can be up to `2^16` operations (262140 bytes of code), the `eip` is 16bit.

Conditionals are handled by saving the 'jump address' into the `jip` special register, then calling jump instructions such as `jmp neq` and `jmp eq` which - if the condition matches - will set `eip` to `jip`, otherwise increment `eip` and continue.


## Example Program

```asm
$ZERO 0   # Friendly names for registers
$ONE 1

:start	jmp to :main
:exit	jmp die
:main
	reg mov $ZERO $ONE
	reg swp $ONE $ZERO
	jmp set :exit
	jmp neq $ZERO $ONE

:end jmp eq $ZERO $ONE

```

When executing it will generate a trace:

```
@0    jmp to 2 0
@2    reg mov 0 1
@3    reg swp 1 0
@4    jmp set 1 0
@5    jmp neq 0 1
@6    jmp eq 0 1
@1    jmp die 0 0
```

The assembler will generate a listing:

```
01010200 @0    jmp to :main   # test.asm:5
01000000 @1    jmp die    # test.asm:6
00000001 @2    reg mov $ZERO $ONE  # test.asm:8
00020100 @3    reg swp $ONE $ZERO  # test.asm:9
01020100 @4    jmp set :exit   # test.asm:10
01040001 @5    jmp neq $ZERO $ONE  # test.asm:11
01030001 @6    jmp eq $ZERO $ONE  # test.asm:13
```

## Example command

```c
static int jmp_eq( nanac_t *cpu, uint8_t arga, uint8_t argb ) {
	cpu->do_jump = cpu->regs[arga] == cpu->regs[argb];
	return 0;
}
```

## Example Usage

```c
int main( int argc, char **argv ) {
	nanac_t cpu;
	nanac_init(&cpu);
	nanac_builtins(&cpu);

	print_mods(&cpu);

	if( argc > 1 ) {
		load_file(&cpu, argv[1]);
	}

	nanac_run(&cpu);

	if( cpu.ops ) {
		free(cpu.ops);
	}

	return 0;
}
```
