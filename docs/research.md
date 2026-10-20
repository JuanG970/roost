# Supervising many coding agents

A survey of research on supervising many agents at once, made on 2026-10-20,
from which the [roadmap](roadmap.md) takes its order and several ideas. Its
sources are linked where they're used. What it says of Roost is as of that
day: since then, `M-x roost-workload` shows how long agents waited on you
([#83](https://github.com/csheaff/roost/pull/83)), the task panel shows your
latest prompt ([#84](https://github.com/csheaff/roost/pull/84)), and a task's
size and spread are in its panel, with a mark on its row from 400 lines, 10
files or 3 top-level directories
([#85](https://github.com/csheaff/roost/pull/85)).

One person can keep about three to five coding agents productive while judging
their work personally, and most people run two to four: the only controlled
trial of a supervision dashboard raised peak use from 2.3 to 3.3 agents
([ParallelPilot](https://arxiv.org/html/2609.33113v1)), and fleets in the tens
appear only where machines do most of the reading
([claude.dev](https://claude.dev/blog/solving-code-review-at-anthropic/)). The
limit is a ratio between how long an agent runs before it needs you and how
long you take to serve it, and serving is mostly noticing, reorienting and
reviewing, not typing ([Olsen &
Wood](https://hci.rwth-aachen.de/materials/conferences/CHI2004/1p231.pdf)).
Supervisory-control models put the largest single loss in agents that wait
before anyone notices and understands them, which cut predicted capacity by up
to 67% ([Cummings & Mitchell
2008](https://doi.org/10.1109/tsmca.2007.914757)); once noticing is fast,
review binds, because careful reading runs near 200 lines an hour ([Kemerer &
Paulk](https://sites.pitt.edu/~ckemerer/PSP_Data.pdf)). For "which agent needs
me", people take in one brightness or colour difference on an otherwise quiet
list fastest, then the first two words of a label; for "is this work good",
nothing glanceable is reliable, and the agent's own summary, the quickest
thing to read, raises acceptance whether or not it is true ([Bansal et
al.](https://arxiv.org/abs/2006.14779)). Roost already follows most of the
evidence on noticing: states from the agents' hooks, a sidebar that stays
blank until something needs you, a waiting count, permission requests first,
notifications that quote the question, and no answering on the agent's behalf.
It departs from the evidence in sending one undifferentiated notification per
event, in letting a glance at a reply clear every sign that a task's work is
unreviewed, and in offering little between a file list and a full read. Beyond
the two changes already in progress, the best-supported changes are the
roadmap's "since you last looked" and evidence beside the files, shaped as
facts rather than verdicts; size marks at the scale where human review
degrades; notifications split by urgency and grouped; and a warning, when a
task starts, that another task already changes its files.

## Measured fleets sit near three, and a dashboard adds about one

The direct evidence on coding agents is thin but points one way. In
ParallelPilot's interviews with 14 developers who ran parallel sessions
routinely, **three at once was the most common setup** (8 of 14), most ran two
to four, and lifetime maxima ranged from three to eight; one person spread
small, isolated issues across 20 cloud sessions
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). Its controlled trial
(16 developers, counterbalanced, 20-minute blocks) put a planner, a run log
and a status dashboard around Copilot CLI and raised **peak concurrency from
2.31 to 3.25 agents**, tickets per minute by 63%, and the number participants
felt they could supervise in future from 2.56 to 4.00
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). Copilot's production
traces show a median of two sessions per user, ninetieth percentile eight,
without saying how many ran at once ([Copilot
traces](https://arxiv.org/abs/2608.00101)). Practitioners report the same
band. Simon Willison says he can review and land only one significant change
at a time
([Willison](https://simonwillison.net/2025/Oct/5/parallel-coding-agents/)),
and Addy Osmani puts his ceiling at three or four threads and prefers three
well-reviewed ones to six half-supervised
([Osmani](https://addyosmani.com/blog/cognitive-parallel-agents/)). The large
operations at Anthropic and OpenAI got past that by moving most reading to
machines and evidence gates, keeping people on direction, risk and the
evidence itself
([claude.dev](https://claude.dev/blog/solving-code-review-at-anthropic/)); in
OpenAI's harness project, "synchronous human attention is the only scarce
resource" ([Latent Space](https://latent.space/p/harness-eng)).

Supervisory-control research on drones and robots gives the same shape with a
sharper distinction: operators can supervise far more vehicles than they can
control. One experienced operator watched up to 15 UAVs efficiently but could
run the mission and payload of only about three, and at four began to neglect
the fourth feed ([Porat et al.
2016](https://pmc.ncbi.nlm.nih.gov/articles/PMC4878290)). In a
search-and-rescue simulation, team score peaked at four to six robots; with
eight, operators did worse than with six because they committed too many
robots early, and only three of them chose to leave robots idle ([Crandall et
al. 2011](https://faculty.cs.byu.edu/~crandall/papers/Crandall_tsma2011.pdf)).
Estimates across the field run from three to twelve depending on autonomy and
task ([Lewis et al. 2010](https://doi.org/10.1177/0018720810366859)). For
coding agents, supervising means glancing at status and answering quick
prompts; controlling means reading diffs, steering design and merging, and
that half binds first.

| Source | Setting | What one person handled |
| --- | --- | --- |
| [ParallelPilot](https://arxiv.org/html/2609.33113v1), interviews (N=14) | Parallel coding agents on their own work | 3 most common; 2–4 typical; maxima 3–8 |
| [ParallelPilot](https://arxiv.org/html/2609.33113v1), trial (N=16) | Coding tickets, without review or merging | Peak 2.31 without the dashboard, 3.25 with it |
| [Willison](https://simonwillison.net/2025/Oct/5/parallel-coding-agents/), [Osmani](https://addyosmani.com/blog/cognitive-parallel-agents/) | Personal parallel coding | 3–4, review-bound |
| [Porat et al. 2016](https://pmc.ncbi.nlm.nih.gov/articles/PMC4878290) | Experienced UAV operators | About 15 supervised, about 3 controlled |
| [Crandall et al. 2011](https://faculty.cs.byu.edu/~crandall/papers/Crandall_tsma2011.pdf) (N=16) | Simulated rescue robots | Score peaks at 4–6; 8 worse than 6 |
| [Cummings & Guerlain 2007](https://pubmed.ncbi.nlm.nih.gov/17315838) (N=42) | Navy personnel, autonomous missiles | Degraded at 16; busy more than 70% of the time predicts decline |

The model underneath is fan-out: one plus the ratio of how long an agent can
be ignored before its performance drops (neglect time) to how long it takes a
person to bring it back to full speed (interaction time) ([Crandall et al.
2005](https://students.cs.byu.edu/~crandall/papers/CGON_SMCa_05.pdf)).
Cummings and Guerlain found performance fell once operators were busy more
than about 70% of the time, a threshold from one 42-person study that is best
read as a rule of thumb ([Cummings & Guerlain
2007](https://pubmed.ncbi.nlm.nih.gov/17315838)). Combining the two gives an
illustrative budget that no study has checked for code: roughly 0.7 × (run
time + service time) ÷ service time agents at once. Agents that run ten
minutes and then need three minutes of reading, answering or review allow
about three at once; agents that run thirty minutes and need fifteen minutes
of review allow about two; the same thirty-minute agents judged in four
minutes from their evidence allow about six. **Longer, more autonomous runs do
not raise the count by themselves, because they produce more to review.** At
about 200 lines an hour of careful reading, a task that writes 300 lines takes
90 minutes to read, which holds someone who reads every line near one agent
unless the tasks are small. That is why every account of reaching three to
five agents comes with small tasks, delegated checks, or both. The arithmetic
also flatters each added agent: Olsen and Wood argue, with some supporting
data, that interaction time itself grows with the number of agents, as
monitoring, choosing and switching take longer ([Olsen &
Wood](https://hci.rwth-aachen.de/materials/conferences/CHI2004/1p231.pdf)).

What is not known matters as much. No study measures one person reviewing and
merging the work of more than five coding agents. ParallelPilot is an
unreviewed preprint whose sessions lasted 20 minutes and left out review and
merging, so its throughput gain is an upper bound, and METR could not time
tasks reliably for developers who ran several agents at once ([METR
update](https://metr.org/blog/2026-02-24-uplift-update/)). Every measured
capacity above five comes from vehicles and robots, where tasks are spatial
and service takes seconds rather than the tens of minutes a review can take;
the coding reports above five, such as 20 cloud sessions, describe small,
isolated tasks.

## Agents lose most of their time waiting to be noticed, then read

The largest loss is waiting to be noticed. Cummings and Mitchell split the
time a supervised vehicle waits into time in interaction, time queued behind
other vehicles, and time before the operator notices and understands the new
situation. Adding these waits to a capacity model parameterised from
experiments cut predicted capacity by **up to 67%**, with lost situation
awareness the largest part, and still by 36% when the system managed by
exception ([Cummings & Mitchell
2008](https://doi.org/10.1109/tsmca.2007.914757)). Coding shows the same leak.
ParallelPilot's participants kept discovering agents that had been waiting on
a question nobody had answered, and clarification requests and errors were the
cues they missed most ([ParallelPilot](https://arxiv.org/html/2609.33113v1));
in Copilot's traces, user idle time is a median 80.1% of a multi-turn
session's wall-clock time ([Copilot
traces](https://arxiv.org/abs/2608.00101)). This part is fixable: with the
dashboard, participants' rating of how well they noticed agents finishing or
needing them rose from 3.63 to 6.19 on a seven-point scale
([ParallelPilot](https://arxiv.org/html/2609.33113v1)).

Noticing pays only if the signal is trusted. Olsen and Wood distinguish an
agent that runs by itself from one that is actually neglected: people who
don't trust it watch it while it runs, and fan-out falls toward one however
capable it is. Their participants redirected robots before they needed it and
gave them shorter goals, so neglect time measured in advance consistently
overestimated what people allowed ([Olsen &
Wood](https://hci.rwth-aachen.de/materials/conferences/CHI2004/1p231.pdf)).
Developers rarely watch agents continuously; they rely on cues such as an
unusually long run ([Dhanorkar et al.](https://arxiv.org/abs/2606.05391)),
hard-interrupt in only about 4.5% to 9% of turns, and push back after a turn
in nearly half ([SWE-chat](https://arxiv.org/abs/2604.20779);
[Anthropic](https://www.anthropic.com/research/measuring-agent-autonomy)). The
weight of supervision therefore sits at turn ends, where a reliable signal
that an agent needs you now lets a person stop watching. When alarms are
unreliable, higher workload lowers trust further, amplifying the damage
([Chien et al. 2013](https://doi.org/10.1177/1541931213571260)).

Interaction time is mostly not typing. Olsen and Wood divide it into
monitoring and choosing the next agent, re-acquiring its goals, solving the
problem, and expressing a command, and with several robots command expression
was a minor part; a worse interface, a smaller screen that forced scrolling,
raised interaction effort by a quarter to a half and cut fan-out for their
most capable robots from about 5.1 to 4.0 ([Olsen &
Wood](https://hci.rwth-aachen.de/materials/conferences/CHI2004/1p231.pdf)).
Re-acquiring a goal costs seconds in the lab, where resumption took 3.8 s
against 1.9 s for an ordinary next action ([Altmann & Trafton
2004](https://escholarship.org/content/qt18b4r661/qt18b4r661_noSplash_3f34d996b3c4b2f3df0252199a2e55fc.pdf)),
but minutes in the field: interrupted office work came back after a mean of 25
minutes, with 2.26 other activities in between ([Mark et al.
2005](https://www.ics.uci.edu/~gmark/CHI2005.pdf)), and only 10% of
programming sessions had an edit within a minute of starting ([Parnin &
Rugaber](https://sites.cc.gatech.edu/reverse/repository/resumptionstrategies.pdf)).
Switching between highly automated tasks takes longer than switching between
manual ones, and having fewer of them does not make it cheaper ([Squire &
Parasuraman 2010](https://doi.org/10.1080/00140139.2010.489969)).

Once noticing is fast, review is where the time goes. It is the most discussed
stage of supervising an agent, 366 of 1,114 coded quotes in one study ([Work
Behind Delegation](https://arxiv.org/html/2609.24234)), and developers
describe re-reviewing everything after each round of fixes to code they did
not write ([Dhanorkar et al.](https://arxiv.org/abs/2606.05391)). Reviews at
or below **200 lines an hour** found 56–66% of defects, faster ones 45–51%
([Kemerer & Paulk](https://sites.pitt.edu/~ckemerer/PSP_Data.pdf)). Larger and
more scattered changes get worse review: each extra file in a pull request
lowered the odds that a given file got any comment by about 8.7% ([Rahman,
Codabux & Roy](https://arxiv.org/abs/2609.22610)), and in Chromium OS,
security defects in changes spread across more directories were missed more
often ([Paul et al.](https://ar5iv.labs.arxiv.org/html/2102.06909)).

Agents that touch the same files stop being independent, and every capacity
model assumes they are. Replayed merges of concurrent agent pull requests
conflicted textually in **19.8% of same-agent pairs and 41.7% of cross-agent
pairs**, a lower bound since build and semantic conflicts went uncounted ([Xu
et al.](https://arxiv.org/pdf/2607.04697)), and 14 of ParallelPilot's 16
participants had run into overlapping or conflicting edits
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). The trial's most
telling result, though exploratory, sits here: gains in coordination support
(seeing dependencies and shared files) tracked objective output (ρ = .55 for
tickets, .62 for throughput), while gains in monitoring support tracked only
how efficient people felt
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). **Better status
displays make supervision feel easier; keeping agents apart and review cheap
is what moves output.**

The person's own load sets the remaining margin. Attention delays grow with
how busy an operator is, so costs compound near the ceiling ([Donmez et al.
2010](https://doi.org/10.1109/tsmca.2010.2046731)), and operators over-commit
rather than leave agents idle ([Crandall et al.
2011](https://faculty.cs.byu.edu/~crandall/papers/Crandall_tsma2011.pdf)).
People interrupted often work faster but report more stress, frustration and
effort after only 20 minutes ([Mark, Gudith & Klocke
2008](https://www.ics.uci.edu/~gmark/chi08-mark.pdf)), the lab version of
Willison's account, as Osmani retells it, of four parallel agents leaving him
spent by late morning
([Osmani](https://addyosmani.com/blog/cognitive-parallel-agents/)).

Finally, knowing that an agent needs you is not knowing whether its work is
right. ParallelPilot improved every tracking measure but not felt control
(3.94 to 4.31, p = .414), trust in the output, or ease of verifying it; nine
of 16 participants saw no difference in verification
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). A better trace view
for a computer-use agent likewise sped up finding errors without making people
more accurate ([Grunde-McLaughlin et al.](https://arxiv.org/abs/2602.16844)),
and in five-hour sessions 94% of participants without a monitor missed
sabotage inserted into their agent's work ([Ye et
al.](https://arxiv.org/abs/2606.05647)). Usage limits are named as a cap too,
but the evidence is vendor guidance that parallel sessions use them up in
parallel, not measurement ([claude.dev cloud
guide](https://claude.dev/blog/claude-code-in-the-cloud/)).

## A coloured word finds the agent; nothing glanceable vouches for its work

### One cue that pops out, then the first two words

An item that differs from its neighbours in a single feature, such as
brightness, hue or motion, is found in about the same time however many items
surround it, while one defined by a combination of features needs a serial
search costing tens of milliseconds per item
([Healey](https://www.csc2.ncsu.edu/faculty/healey/PP/)). Brightness dominates
hue, and searching for a known target with a fixed colour code is barely
affected by how many colours are on screen, whereas finding the odd one out
and judging how many kinds are present degrade as scattered hues multiply
([Healey](https://www.csc2.ncsu.edu/faculty/healey/PP/); [Haroz &
Whitney](https://whitneylab.berkeley.edu/PDFs/Haroz_Whitney_2012_InfoVis.pdf)).
That makes a quiet list with a few coloured rows the right shape. In a
dual-task study, learned coloured squares on a second monitor were read
without a direct look **90% of the time with three codes and 76% with seven;
text 5% and 0%**. Text was still the fastest code to interpret when unfamiliar
and the least disruptive to the main task ([Matthews et al.
2006](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2006/EECS-2006-173.pdf)).
Colour says that something needs you, and the word says what, once you look.

Colour alone is weak far from where the eyes are: about a quarter of colour
changes far in the periphery were missed, against under 2% of small motions,
and among motions a slow blink distracted least while zooming and travelling
distracted most ([Bartram, Ware & Calvert
2003](https://www.interruptions.net/literature/Bartram-IJHCS03.pdf)). Changes
made while a person looks elsewhere often go unseen altogether ([Rensink et
al.
1997](https://homepage.villanova.edu/diego.fernandezduque/Teaching/CognitivePsychology/Lectures_and_Labs/s3Attention/sReading/Rensink_etal_97.pdf)),
so a waiting state has to persist until dealt with and be summed in a count.
About four items can be held in mind or counted at a glance ([Cowan
2001](https://pubmed.ncbi.nlm.nih.gov/11515286/)); beyond four waiting agents
an exact number beats eyeballing. In lists, people read about the first two
words, or eleven characters, of each item
([NN/g](https://nngroup.com/articles/first-2-words-a-signal-for-scanning)).

### Notifications have a budget, and urgency has to stay honest

Process-control guidance puts the alarm rate a dedicated operator can very
likely accept at about one per ten minutes and the most that is manageable at
about two ([ISA-18.2, via
Tennant](https://www.yokogawa.com/ro/library/resources/media-publications/implementing-alarm-management-per-the-ansi-isa-182-standard-control-engineering/)),
with a flood at ten in the ten minutes after an upset
([HSE](https://www.hse.gov.uk/pubns/chis6.pdf)). These are survey-based norms,
not measured limits, and a developer who is also doing their own work will
likely tolerate fewer. Five agents that each finish a turn every ten minutes
already make five events per ten minutes. People learn each channel's
reliability: about 90% of 138 participants matched their response rate to how
often an alarm turned out to be true ([Bliss, Gilson & Deaton
1995](https://stars.library.ucf.edu/scopus1990/1980)), and nurses responded
more slowly the more non-actionable alarms they had heard in the previous two
hours ([Bonafide et al.](https://www.pubmed.ncbi.nlm.nih.gov/25873486/)). A
channel that carries both "blocked until you answer" and "finished, look when
convenient" teaches people the mixed rate and slows them on the urgent kind.

The standards answer with a strict rule, that an alarm must call for an
operator action and a status indicator is not an alarm
([HSE](https://www.hse.gov.uk/pubns/chis6.pdf); [Control
Global](https://www.controlglobal.com/protect/safety-instrumented-systems/article/11382017/alarm-management-asm-alarm-management-guidelines-and-isa-182-how-do-they-stack-up-control-global)),
and with the cockpit's three levels, defined by how soon the crew must know
and how soon act ([14 CFR
25.1322](https://www.law.cornell.edu/cfr/text/14/25.1322)): one aural alert at
a time, and an overflow message giving the number and urgency of the rest
([FAA human factors
guidance](https://hfcc.dot.gov/publications/docs/GeneralGuidance/zz_FAA_GeneralGuidanceDoc_Chapter_04_Section_01.html)).
Timing matters as well. McFarlane found that announcing an interruption and
letting the person choose when to take it was best overall, and immediate
delivery best only for promptness ([McFarlane
2002](https://www.interruptions.net/literature/McFarlane-HCI02_2.pdf));
holding notifications until the next breakpoint cut frustration by 20% and
reaction time by 25% at an average delay of about 90 seconds, though the
system's guesses at breakpoints matched users' only about 40% of the time
([Iqbal & Bailey
2010](https://www.interruptions.net/literature/Iqbal-TOCHI10.pdf)). Batching
phone notifications to three times a day lowered stress, while turning them
off raised anxiety ([Fitz et al.
2019](https://scholars.duke.edu/publication/1402953)). The pattern the
evidence supports is fewer, grouped alerts plus a reliable count that is
always there, not silence.

### Coming back needs the place where work stopped

On return, the best cue is the stopping point. Programmers who resumed in the
method they had last edited started editing within a minute 35% of the time,
against 10% overall ([Parnin &
Rugaber](https://sites.cc.gatech.edu/reverse/repository/resumptionstrategies.pdf)),
and in the lab, automatic cues showing recent activity in time order **about
doubled task success** over the developer's own notes ([Parnin & DeLine
2010](https://www.microsoft.com/en-us/research/publication/evaluating-cues-for-resuming-interrupted-programming-tasks/)).
A clear signal that an interruption had ended raised resumption of the
suspended task from 52% to 90% ([Dodhia & Dismukes
2009](https://interruptions.net/literature/Dodhia-ApplCognitPsychol09.pdf)).
Interruptions of three to four seconds doubled to tripled place-keeping errors
on the next step ([Altmann et al.
2014](https://www.interruptions.net/literature/Altmann-JExpPsycholGen14.pdf)),
which makes stepping through a diff or a merge the worst moment for a
notification.

### Judging the work needs checkable facts, not the agent's account

The fast signals of quality are objective and partial. Size, files and spread
predict outcomes: unmerged agent pull requests are larger, touch more files
and fail CI more often ([Ehsani et al.](https://arxiv.org/abs/2601.15195)),
and practitioners name passing CI as the gate for reviewing an AI-authored
pull request (66.5%) and testing evidence as the main sign of credibility
(81.2%) ([Rahman et al.](https://arxiv.org/html/2610.11179v1)). The agent's
own summary is the fastest thing to read and the least reliable. Inaccurate
self-reporting, such as claiming a success that did not happen, made up 22.58%
of misalignment episodes in 20,574 sessions, a growing share ([How Coding
Agents Fail Their Users](https://arxiv.org/abs/2605.29442)); explanations
raised acceptance of AI advice whether or not it was right ([Bansal et
al.](https://arxiv.org/abs/2006.14779)); and programmers caught only 49% of
incorrect LLM-written assertions, with the same confidence they had when right
([Kaufman et al.](https://arxiv.org/html/2607.08885)). What reduced
overreliance was making verification cheaper than doing the task ([Vasconcelos
et al. 2023](https://arxiv.org/abs/2212.06823)).

Aids that recommend hurt more than aids that inform when they are wrong.
Across 18 experiments, the sharpest losses came when automation crossed from
analysing information to choosing actions ([Onnasch et al.
2014](https://depositonce.tu-berlin.de/items/7df9879f-d77b-4f2b-8760-f0e3d174ca67/full)),
and in clinical systems wrong advice raised the risk of a wrong decision by a
quarter (RR 1.26), less so where systems presented information rather than
commands ([Goddard et al.
2012](https://pmc.ncbi.nlm.nih.gov/articles/PMC3240751)). Two effects grow as
agents improve. Complacency appears when monitoring competes with other work
([Parasuraman, Molloy & Singh
1993](https://doi.org/10.1207/s15327108ijap0301_1)), and misses climb as
problems get rare, from about 7% at 50% prevalence to about 30% at 1% ([Wolfe
et al. 2005](https://doi.org/10.1038/435439a)). Staying active protects
understanding: in a randomized trial, the AI-assisted group trailed by 17
points on later comprehension, most on debugging, and those who delegated
without trying to understand scored lowest ([Shen & Tamkin
2026](https://arxiv.org/abs/2601.20245)).

## Roost already gets noticing right and stops short at judging

Roost's design for "which agent needs you" reads like an application of the
alarm and glanceability literature. Its most important choice is the least
visible: states come from each agent's own lifecycle hooks, checked against
the agents themselves with `make contract` and against recorded sessions in
`make test`, rather than from a model reading transcripts, as ParallelPilot's
triage did. The reliability of that signal is what lets someone stop watching,
and no study in these notes measured how accurate model-inferred cues are. The
table compares the evidence with Roost as the manual and the code describe it
today.

| What the evidence supports | Roost today | Fit |
| --- | --- | --- |
| A trustworthy "needs you" signal, or people go back to watching | States from each agent's hooks, checked by `make contract` | Matches |
| Show nothing for a task that needs nothing; status is not an alarm | A seen agent that waits for nothing has no status word; working and stopped rows are dim | Matches |
| Few hue families, brightness for the first split, a fixed mapping | Needs-you statuses fold into four hue families over dim and plain text | Matches |
| Persistent state and a count, against change blindness | The word stays until dealt with; `Roost:2` in the mode line and "2 waiting" in the sidebar, coloured as the most urgent | Matches |
| Alerts that say which, what and what next, without seizing focus | Title names task and status; the body quotes the request or the reply's first line; nothing takes focus; the task on screen doesn't notify | Matches, though the title's first characters are "Roost:" |
| Blocking items first; the user picks, the system suggests | `n` visits permission requests and startup prompts first; it is a key you press, and any row opens | Matches |
| One alert for a shared cause | An unreachable host is one line; its tasks keep their last state, marked offline | Matches |
| A delay before fleeting states alarm | A failed check or new review thread waits two minutes before it counts | Matches, for pull requests only |
| Cues at the stopping point on return | Panel: next step, original prompt, changed files, latest reply; your latest prompt is in progress | Mostly matches |
| Isolation and early notice of interference | A worktree per task; `conflicts` from `git merge-tree`; the panel names tasks changing the same files | Matches once files change; nothing before a task starts |
| Active review; merge decisions stay human | Staging as approval, `;` notes, `t` shell; `m` and `P` only on your command | Matches |
| A stalled agent is a cue people rely on | Dashboard and panel show "for 14m" since the agent's last event, dim; the sidebar doesn't | Partial |
| Separate urgency levels; group bursts; keep within a rate | One notification per change to ready, permission, failed, crashed or exited, the same form for all; replaced per task, never grouped across tasks | Contradicts at four or more agents |
| A reported finish is not an accepted one; acknowledging is not resolving | Seeing a ready agent's reply blanks its row, though its work may be unreviewed and unmerged | Partly contradicts |
| Review degrades past a few hundred lines, about ten files, several directories | No size or spread in the sidebar or panel; the roadmap's large-change mark is at 1,000 lines | Contradicts in its threshold |

Three departures stand out. The first is notifications. Each change to a
waiting state sends its own notification, and terminal-notifier replaces an
older one for the same task, but nothing groups them across tasks, and an
agent blocked on permission looks like one that finished. At two agents this
costs little. At five, the stream exceeds what process control calls
manageable for an operator with no other work, and probability matching
predicts slower responses to the permission requests that matter most. The
second is "seen". Looking at a ready agent's reply blanks its sidebar row,
which is the right acknowledge-and-reset behaviour for the alert ([FAA human
factors
guidance](https://hfcc.dot.gov/publications/docs/GeneralGuidance/zz_FAA_GeneralGuidanceDoc_Chapter_04_Section_01.html)),
but the condition that remains, finished work nobody has reviewed or merged,
then has no mark in any view you can glance at. ParallelPilot kept an agent's
reported finish separate from the developer's acceptance for exactly this
reason ([ParallelPilot](https://arxiv.org/html/2609.33113v1)). The third is
review support, which stops at a file list and Magit. The roadmap's planned
mark for large changes at 1,000 lines borrows the size at which Anthropic's AI
reviewer usually finds something
([claude.dev](https://claude.dev/blog/solving-code-review-at-anthropic/)), not
the size at which human review degrades, which the evidence puts at a few
hundred lines, about ten files, or several directories.

Two candidates in the roadmap's "Worth trying" and "Exploring" lists also cut
against the review ceiling: several attempts at one prompt, and agents that
start Roost tasks, both multiply output without making it cheaper to judge.
The evidence fits them only where comparing attempts is cheap or a person
confirms the split before agents start, as in ParallelPilot, where reviewing
the planner's split before any agent started was part of the workflow the
trial tested. The roadmap also still lists pull requests that wait on you and
conflicts before merge time as ideas, though the manual and changelog describe
both as working.

## The best-supported changes shorten review, not the walk to it

The changes below are ranked by evidence and expected effect together, and
each carries a tag for its evidence. **Strong** means field data, a
meta-analysis or several controlled studies pointing the same way, with close
transfer to reviewing code. **Moderate** means one or two controlled studies,
or consistent standards and practice, carried over from another supervision
domain. **Weak** means a single small study, a simulation, one company's
practice or practitioner reports. Because monitoring gains in ParallelPilot
tracked how efficient people felt rather than what they got done, in an
exploratory analysis, changes that shorten review and keep agents independent
rank above further work on noticing, which Roost already does well. All of
them stay inside Roost's rules: none answers a prompt, rebuilds an agent's
menus, or approves or merges anything; Roost marks and counts, and the user
decides.

The two changes already being built are well founded. Showing your most recent
prompt in the task panel supplies the cue that memory-for-goals research says
resumption depends on, the goal you last set rather than the one you started
with, and it is the agent-task form of the last-edit cue ([Altmann & Trafton
2002](https://www.interruptions.net/literature/Altmann-CogSci02.pdf); [Parnin
&
Rugaber](https://sites.cc.gatech.edu/reverse/repository/resumptionstrategies.pdf))
**[Strong for the mechanism]**. Logging status changes to report how long
agents waited on you measures what the capacity models count ([Cummings &
Mitchell 2008](https://doi.org/10.1109/tsmca.2007.914757)) **[Moderate]**. The
evidence says what that log should keep apart: the time from an agent starting
to wait until you first look (the noticing wait, the largest loss in the
models), the time from that look until the agent works again or the task
merges (service), the length of agents' unattended runs (neglect time), and
notifications per hour against the norm of one or two per ten minutes. With
run and service times per person, fan-out stops being a borrowed number and
becomes each user's own. Counts per task would also expose the "bad actor"
that alarm audits find, as in one plant where nine alarms produced 85% of all
alarm activity ([Energy
Institute](https://hpog.org/assets/documents/BN-2-Alarm-handling-web.pdf)): an
agent that asks permission for the same tool all day is fixed in that agent's
own settings, by the user. The report should show recorded facts, not how the
day felt, since developers' sense of their own speed has run opposite to
measurement ([METR RCT](https://arxiv.org/abs/2507.09089)).

| Rank | Change | Where it stands | Evidence | Expected effect |
| --- | --- | --- | --- | --- |
| 1 | Since you last looked, by content, the agent's own changes included, on the row, the panel, `r`, `m` and `P` | Roadmap, "most likely first" | Moderate | High: cuts re-review, the most discussed cost |
| 2 | Evidence beside the files as facts: checks by where they ran, with none reported shown apart from passed, findings with file and line, the reply labelled as the agent's account; no badge or score | Roadmap, refined | Moderate to strong | High on judgment; modest on speed |
| 3 | Size and spread in the panel, and a dim mark on the row past about 400 lines, 10 files or several top-level directories, beside risky paths | Roadmap, threshold changed | Strong in direction, weak for the exact numbers | Medium to high |
| 4 | Notifications by urgency: blocking requests alone and at once; finished, failed and pull-request turns grouped across tasks after a short delay | New | Moderate | Small at two or three agents, large at five or more |
| 5 | At task start, name running tasks that already change the files the draft points at | New, beside the roadmap's file-buffer header line | Moderate | Medium: keeps agents independent |
| 6 | `n` says where it went and why; an empty queue says so and offers the way back | New | Moderate for the end signal, weak for the rest | Low to medium, cheap |
| 7 | A dim "quiet" word in the sidebar for a running task far past its usual gap between events, not counted as waiting | New | Moderate | Low to medium |
| 8 | A local check command that must pass before `m` | Roadmap, "Scripts beyond setup" | Moderate | Medium for local merges |
| 9 | Hold non-blocking notifications while you review, merge or write a prompt in another task, a few minutes at most | New | Moderate in the lab, untested with Roost's events | Medium, at some complexity |
| 10 | How often each agent's or kind of task's merged work was reverted or fixed soon after | New | Weak | Uncertain |
| 11 | Risky-first file order and visited marks in review; a count of note-and-revise rounds | New | Weak | Low |

**Since you last looked** ranks first because it shortens the re-review after
every fix that developers single out in post hoc review, the form of oversight
they discuss most ([Dhanorkar et al.](https://arxiv.org/abs/2606.05391)) and
is the chronological recent-activity cue that doubled task success in the lab
([Parnin & DeLine
2010](https://www.microsoft.com/en-us/research/publication/evaluating-cues-for-resuming-interrupted-programming-tasks/)).
The roadmap records the commit at `r` or when notes are sent; the evidence
suggests three refinements. Compare by content, uncommitted work included,
because agents amend and rebase between looks, as Google's review tool
compares any two snapshots ([Sadowski et
al.](https://sback.it/publications/icse2018seip.pdf)). Count everything the
agent changed, not only its answers to your notes, since 61–81% of changes
made during code review were not prompted by a reviewer's comment ([Fregnan et
al.](https://pmc.ncbi.nlm.nih.gov/articles/PMC9489594/)). And show the count
dimly on the row of a task you have seen, such as "+3 commits", which repairs
the second departure without making unreviewed work count as waiting. Staging
stays the finest record of approval, and "3 hunks changed since you staged"
says the same thing at hunk level. This serves both kinds of user: those who
read every diff re-read only what changed, and those who judge by evidence see
when the evidence they judged has gone stale.

**Evidence beside the files** should be facts, ordered by how easily they can
be checked: first the pull request's checks or a local check command's result,
each saying which it is, with "no CI results" looking different from "passed";
then open reviewer findings with file and line; then the agent's latest reply,
labelled as its account. A single badge, score or "ready to merge" line would
move Roost from informing to recommending, where wrong aids cost most
([Onnasch et al.
2014](https://depositonce.tu-berlin.de/items/7df9879f-d77b-4f2b-8760-f0e3d174ca67/full);
[Goddard et al. 2012](https://pmc.ncbi.nlm.nih.gov/articles/PMC3240751)).
Roost need not parse an agent's claims to show which rest on its word alone:
"no CI results" placed beside a reply saying the tests pass tells the reader
the claim is unverified, not that it is false, since the agent may have run
them in its terminal. Findings should point people at hunks worth reading
rather than replace the file list, since automated review comments narrowed
where reviewers looked and turned up more minor issues but no more serious
ones ([Tufano et al.](https://arxiv.org/abs/2411.11401v1)).

**Size and spread** rests on the most consistent finding in review research,
that smaller changes are reviewed better and faster ([Sadowski et
al.](https://sback.it/publications/icse2018seip.pdf); [Bosu, Greiler &
Bird](https://www.microsoft.com/en-us/research/?p=168510); [Kemerer &
Paulk](https://sites.pitt.edu/~ckemerer/PSP_Data.pdf)), with directories
touched a better warning than lines alone ([Paul et
al.](https://ar5iv.labs.arxiv.org/html/2102.06909)). Lines added and removed,
files, and top-level directories belong in the panel; the thresholds for a
mark are heuristics, so they belong in an option. A diff reader then knows
before opening Magit whether a task is ten minutes of reading or an hour, and
the roadmap's risky paths from CODEOWNERS or `.dir-locals.el` say which hour.

**Notifications by urgency** keeps a distinct notification, sent at once and
alone, for an agent that cannot go on without you, and gathers everything else
(finished turns, failures, pull requests that wait on you) for seconds to a
minute into one notification that names the most urgent and counts the rest,
the cockpit's overflow message ([FAA human factors
guidance](https://hfcc.dot.gov/publications/docs/GeneralGuidance/zz_FAA_GeneralGuidanceDoc_Chapter_04_Section_01.html)).
Permission requests must never wait, since a blocked agent only idles. Titles
should lead with the task's name and status, since people read about the first
two words. The mode-line count stays as the signal that is always there, which
the batching studies say must not go away ([Fitz et al.
2019](https://scholars.duke.edu/publication/1402953)). The benefit grows with
the fleet: at two agents it is small, and at five it decides whether the
permission channel stays credible.

**A collision warning at task start** is the cheap, agent-neutral slice of
ParallelPilot's planner, the component whose gains tracked output
([ParallelPilot](https://arxiv.org/html/2609.33113v1)). When a draft carries a
file, as one started from a region, a compilation error, a diagnostic or a
file does, the draft names running tasks of the project that already change
it, from the overlap data the panel already uses. Pre-write claims on files
raised integration success among parallel agents from 65.6% to 96.7%, mostly
by making them work in turn ([Claim Plane](https://arxiv.org/abs/2608.00947));
telling the person before the agent starts gives them that choice without
Roost planning anything.

The rest are smaller. Having `n` say where it went and why ("permission ·
ledger: Asks to run make test; 2 more waiting") answers the reorientation cost
of being sent somewhere you did not choose, which added about 1.6 s per
service for robots ([Crandall et al.
2011](https://faculty.cs.byu.edu/~crandall/papers/Crandall_tsma2011.pdf)), and
an empty queue that says so and offers a return to where you left from is the
end-of-interruption signal that raised resumption from 52% to 90% ([Dodhia &
Dismukes
2009](https://interruptions.net/literature/Dodhia-ApplCognitPsychol09.pdf)). A
dim "quiet" word supplies the one ParallelPilot cue Roost lacks in the
sidebar, an unexpected delay; slow, hanging or excessive behaviour accounts
for 17% of the times people interrupt Claude
([Anthropic](https://www.anthropic.com/research/measuring-agent-autonomy)),
and comparing each task to its own usual gaps avoids flagging long test runs.
A check command before `m` gives local merges the evidence a pull request's CI
gives, and catches the build and semantic conflicts that `git merge-tree`
cannot ([Xu et al.](https://arxiv.org/pdf/2607.04697)). Holding non-blocking
notifications while you are in Magit or a merge for another task protects the
place-keeping work that brief interruptions disrupt most ([Altmann et al.
2014](https://www.interruptions.net/literature/Altmann-JExpPsycholGen14.pdf));
Roost sees those moments directly rather than guessing them, which was the
weak part of the lab system, but nobody has tested that. Revert and
follow-up-fix rates per agent would apply knowledge of results, the
best-supported countermeasure in vigilance research, and people who
experienced automation failures checked more afterwards ([Bahner, Hüper &
Manzey 2008](https://doi.org/10.1016/j.ijhcs.2008.06.001)), but whether such
rates calibrate trust in agents' code is unknown. File order by risk and
visited marks come from one preprint and a survey ([Rahman, Codabux &
Roy](https://arxiv.org/abs/2609.22610)), and the three-round limit on
revisions from one company's practice
([claude.dev](https://claude.dev/blog/solving-code-review-at-anthropic/)).

The evidence also says what not to build. A key that picks and opens the next
agent with no explanation raised reorientation time and was the mode two
thirds of operators liked least ([Crandall et al.
2011](https://faculty.cs.byu.edu/~crandall/papers/Crandall_tsma2011.pdf)).
Predicted finish times are out: operators given probabilistic schedule
predictions did worse than those with no decision support ([Cummings &
Mitchell 2005](https://doi.org/10.2514/6.2005-7060)), and agents' run times
are highly uncertain. Generated recaps were ParallelPilot's lowest-rated
component (5.09 of 7, against 6.88 for the live status view)
([ParallelPilot](https://arxiv.org/html/2609.33113v1)), and agents' accounts
of themselves are increasingly inaccurate, so quoting the agent and linking to
the diff is better supported. A "safe to merge" verdict, merging on green, an
approval inbox and blanket confirmations before every merge all fail on the
evidence too: the first two automate the decision stage, the third takes the
choice from the user, and forcing functions work but are disliked, so they
belong only on high-risk merges ([Buçinca et al.
2021](https://arxiv.org/abs/2102.09692)). A cap on agents has no
evidence-based number; showing how long agents wait, as the log in progress
will, is the defensible form.

## Conclusion

The question "how many agents can one person supervise" turns out to be the
wrong unit. The evidence measures a review budget: how many minutes of
judgment each agent's output costs against how long it runs alone. Noticing
which agent needs you is mostly in place in Roost, apart from how its
notifications scale, and further polish there will mostly make supervision
feel easier. What moves the real limit is the cost of judging each change and
the independence of the tasks, so Roost's next gains lie on the far side of
`n`, in what changed since you last looked, what the change rests on, how big
it is, and whether another task is already in its files. Serving people who
read every diff and people who judge by evidence turns out to be the same
work, because both need to know what changed, how much, and where to look
first.

The measurement already being built matters more than it looks. Every capacity
figure above five agents is borrowed from drones and robots, and the one
coding study stopped before review. A Roost that records how long agents run,
how long they wait to be noticed, and how long they take to serve can give
each person their own limit instead of a borrowed band, and can show whether a
change actually raised output rather than comfort. The danger runs the other
way as agents improve: rarer failures, steadier apparent reliability and
divided attention are the conditions under which people stop checking, so the
tool that lets someone run more agents has to keep the facts in front of them,
keep the decision theirs, and keep them reading.
