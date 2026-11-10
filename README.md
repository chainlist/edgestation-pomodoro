# Pomodoro

A focus timer for [EdgeStation](https://github.com/chainlist/edgestation) that alternates focus
sessions and breaks.

- **Play / Pause** starts or pauses the current phase.
- **Reset** puts the current phase back to its full length.
- **Skip** moves to the other phase.

When a phase ends, the next one waits for a tap to start. Each placement keeps its own timer, which
keeps running while its page is off screen and while EdgeStation is closed.

## Settings

| Setting      | Choices                  | Default    |
| ------------ | ------------------------ | ---------- |
| Focus length | 15, 25, 45 or 50 minutes | 25 minutes |
| Break length | 5, 10 or 15 minutes      | 5 minutes  |
