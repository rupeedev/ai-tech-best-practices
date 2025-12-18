# Claude Code and Git Worktrees

## Scaling Agentic Coding with Parallel Claude Code

[![Kenji](https://miro.medium.com/v2/resize:fill:64:64/1*T6FxT_PZ6kuxcJTd38Y9VQ.png)](https://medium.com/@kenji-onisuka?source=post_page---byline--aa4c41e9faf9---------------------------------------)

[Kenji](https://medium.com/@kenji-onisuka?source=post_page---byline--aa4c41e9faf9---------------------------------------)

**Follow**

3 min read**·**

Jul 15, 2025

186

**1**

![](https://miro.medium.com/v2/resize:fit:1218/1*97dTcZupaZmrz_hiPQEycw.png)

https://devdynamics.ai/blog/understanding-git-worktree-to-fast-track-software-development-process/

**W**hen I first experimented with Claude Code alongside my favorite IDE, I noticed something. While a single agent could tackle complex tasks, I often wished for multiple “opinions” on a design or implementation.

 What if I could generate different takes on the exact specification, side by side? That is actually documented in** **[*Anthropic’s Claude Code best practices*](https://www.anthropic.com/engineering/claude-code-best-practices) — it’s about using Git worktrees — and it has reshaped how I approach non-deterministic AI workflows.

### Why Parallel Agents Matter

Claude Code breaks down engineering tasks into clear plans. Yet, because large language models produce varied outputs, running the same prompt twice can yield distinct results. Rather than fight that variability, we can embrace it — launching multiple agents in parallel and selecting the version that works best.

> “I’d rather review three implementations than roll the dice on a single outcome.”

However, a simple** **`git checkout` approach won’t work: all agents would compete for the same branch, causing merge conflicts and lost progress. That’s where Git worktrees come in.

## Git Worktrees Explained

Worktrees provide physical isolation for each agent:

* **Setup** : From your main directory create a sibling folder called** **`trees`.

```
git worktree add -b task-1 ../trees/task-1
```

Repeat for as many agents as you need (e.g.,** **`task-2`,** **`task-3`).

* **Advantages** : Each agent works independently — you can update ports, install dependencies, and run servers (e.g., on ports 3001–3003) without interfering with one another. Once they finish, merge via standard Git.

Automate worktree creation with this** **`create_worktree.sh` script:

```
#!/bin/bash
BRANCH=$1
mkdir -p ../trees
git worktree add -b $BRANCH ../trees/$BRANCH
cd ../trees/$BRANCH
echo "Worktree created for $BRANCH"
```

A Claude command can streamline this further: it creates worktrees, copies** **`.env` files, updates configs, and preps each environment. Then fires off the detailed plan (your task specifications) to each agent on Claude 4 simultaneously.

### Parallel Execution in Action

Let’s see how this works on a generic front-end project —** ** **DemoApp** :

**Plan:** ****Define the task clearly. For DemoApp, the goal might be: “Convert the main dashboard to a compact, information-dense layout with collapsible panels and updated typography.”

**Agents:** ****Spin up three parallel agents in separate worktrees:

```
for i in {1..3}; do
  git worktree add -b task-$i ../trees/task-$i
done
```

**Runtime:** Each agent consumes tokens and completes. Because each worktree is isolated, they can install dependencies, run local servers (e.g., on ports 3001, 3002, 3003), and apply changes without conflict.

**Testing:** ****Launch all three instances at once:

```
./start-tree-clients.sh 3001 3002 3003
```

In Claude Code, define a custom slash command in .claude/commands/exe-parallel.md:

```
Execute the plan in parallel agents: for i in {1..3}; do cd ../trees/ui-rewrite-$i && claude --model opus-4 "/plan: Implement UI revamp"; done
```

Run with /exe-parallel.

This leverages LLMs’ probabilistic nature: Different agents produce unique perspectives, ideal for ambiguous tasks like UI design.

### Merging and Strategic Use

Post-execution, review summaries. Commit changes per worktree, then merge the best (e.g., the modern version) into main: git merge task-rewrite-2.

Resolve conflicts as usual. Use this bash snippet to merge safely:

```
git checkout main
git merge task-rewrite-2 --no-ff -m "Merge best UI revamp from agent 2"
git push origin main
```

Use this for:

* Ambiguous/hard tasks.
* When failures are likely — multiple agents increase success odds.
* With clear plans, iterative prompting suits single branches.

This isn’t cheap (high token costs), but it maximizes compute: Generate n futures, pick the best, ship faster. As agentic tools like Claude Code evolve, parallel workflows position you to orchestrate, not just code.

In 2025’s AI landscape, scaling agents isn’t optional — it’s essential.

[Artificial Intelligence](https://medium.com/tag/artificial-intelligence?source=post_page-----aa4c41e9faf9---------------------------------------)

[Technology](https://medium.com/tag/technology?source=post_page-----aa4c41e9faf9---------------------------------------)

[Innovation](https://medium.com/tag/innovation?source=post_page-----aa4c41e9faf9---------------------------------------)

[Productivity](https://medium.com/tag/productivity?source=post_page-----aa4c41e9faf9---------------------------------------)

[](https://medium.com/tag/programming?source=post_page-----aa4c41e9faf9---------------------------------------)
