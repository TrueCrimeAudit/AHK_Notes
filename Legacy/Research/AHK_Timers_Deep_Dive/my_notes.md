# Personal Notes - AHK Timers Research

## Key Takeaways
- v2 requires function objects, not string labels like v1
- .Bind() is crucial for class methods - preserves 'this' context
- Negative periods = one-time timers, positive = recurring
- Timer precision limited to ~15ms by OS

## Important Code Patterns

### Essential Timer Class Pattern
```autohotkey
class MyTimer {
    __New() {
        this.timerFunc := this.Execute.Bind(this)
    }
    
    Start() { SetTimer(this.timerFunc, 1000) }
    Stop() { SetTimer(this.timerFunc, 0) }
    __Delete() { this.Stop() }  // Cleanup
    
    Execute() {
        // Timer logic here
    }
}
```

### Self-Deleting Timer Pattern
```autohotkey
DelayedAction() {
    // Do something
    SetTimer(, 0)  // Delete self
}
SetTimer(DelayedAction, -2000)  // Run once after 2s
```

## Questions for Further Research
- [ ] How do timers interact with Critical sections?
- [ ] Performance comparison with Loop+Sleep for high-frequency operations
- [ ] Best practices for timer coordination in complex applications

## Integration Plan
1. Update `/Methods/` with timer examples
2. Create timer pattern templates in `/Templates/`
3. Add timer debugging utilities to `/Snippets/`

## Gaps Identified
- Need more info on timer thread priorities
- Advanced coordination between multiple timers
- Memory usage patterns with many concurrent timers

## Next Research Tasks
1. GUI event handling and timer coordination
2. File system monitoring with timers
3. Network operations and timeout patterns