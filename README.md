# rust_goto_function
an attempt at a full (or as much as rust lets us) goto implementation.

# the idea
Rust allows for breaking out of scoped loops. In terms of how llvm sees it thats just a jump statment which is what we want here.
so if we want to go back to a point say

```rust

'begin:
...
goto begin
...
```

we can implement it like this:

```rust 
'begin: loop{
	...
	continue 'begin;
	break 'begin;
}
...
```

simlarly this sort of goto
```rust
...
goto 'end;
...
'end:
...

```

could be implemented with:
```rust
'pre_end: loop{
...
break 'pre_end;
...

break 'pre_end;
}
'end:
...

```

both methods have issues with loop boundries. specifcly a goto like so
```rust
loop{
	...
	'begin:
	...
	loop{
		...
	goto begin
	}
}
...
goto begin
```

would run into some issues with this method


```rust 
'outer: loop{
	...
	'begin: loop{
		...
		loop{
			//goto begin
			continue 'begin;
			break 'begin;
		}
	}//we need to close begin before outer closes
}
...
continue 'begin; //makes no sense here...
```
even if such a thing could somehow work the borrow checker rules become much more complex here.
since now things after 'outer closes still need to be valid because of the goto.
