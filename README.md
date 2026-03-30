# GuidedTrack Skill

`guidedtrack-skill` is an AI coding agent skill for creating, running, and debugging [GuidedTrack](https://guidedtrack.com) (GT) programs.

Agents using the skill should be able to:

- write GT programs in local `.gt` files
- run [`gtlint`](https://github.com/jrc03c/gtlint) to lint and format `.gt` files
- drive a web browser to create, run, and debug programs on [guidedtrack.com](https://guidedtrack.com)

> **NOTE:** ⚠️ This skill has been developed for and exclusively tested with Claude Code. All subsequent documentation assumes that you're using Claude Code.

# Installation

Run these slash-commands in a Claude Code session:

```
/plugin marketplace add jrc03c/guidedtrack-skill
/plugin install guidedtrack-skill@jrc03c-guidedtrack-skill
```

# Usage

To load the skill, simply mention GuidedTrack in conversation with Claude, or use the `/guidedtrack-skill` slash-command. Then tell Claude what you want done.