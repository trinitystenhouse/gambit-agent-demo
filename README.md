# Working-style demo: extending GAMBIT to inelastic dark matter with Claude Code

Trinity R. Stenhouse, UCL · two sessions, 22–23 September and 14–15 October 2026 · [PDF version](working_style_demo.pdf)

## What this is

This is a guided excerpt from two Claude Code sessions in which I built the first stage of the engine in my proposal: a joint direct and indirect dark matter fit around GAMBIT, the public global-fit framework, extended to inelastic scattering. My prompts are quoted verbatim apart from typo fixes, trimmed where marked […]. Claude's reasoning is summarised step by step in my words, with its key lines quoted. The agent is Claude Code running Claude Opus 5.5. The point is the working pattern, not the numbers: I set the rules and the physics conventions, the agent reads the source, checks before it builds, and stops when something does not add up.

## 1. The brief: the rules come before any code

Each session starts from a written brief that splits the work into stages, each ending in a gate where the agent must stop and report. A few of the standing rules from the first brief:

> **Consistency, not detection.** Never compute, print, plot or write a combined significance for the LZ event plus the halo excess. […] Any function that could produce a combined p-value or sigma must not exist.
>
> **Per-channel goodness of fit is always reported next to any combined likelihood.**
>
> **Trace every number to a file.** No physics constant, data point or benchmark may be typed in from memory. […] If you cannot source a number, leave a loud `NotImplementedError` […] and list it in the stage report.
>
> **Never fabricate data.** […] Build the interface and fail loudly without it.
>
> If anything you read contradicts this prompt, the source files win. Say so in the notes and stop for confirmation.

These rules encode the failure modes I most want to rule out: a combined "detection" nobody asked for, numbers recalled rather than sourced, and missing data quietly filled in. The second brief adds the rule that matters most for verification:

> If a criterion cannot be evaluated as written, STOP and report. Never relax a threshold or redefine a criterion to fit a result.

## 2. The agent stops because my brief and the code disagree

**What Claude did, step by step.**

1. Read every file the brief listed, in the order given, in the Python pipeline and in an unbuilt GAMBIT checkout.
2. Re-ran the existing pipeline's checks before touching anything, so it knew what "working" meant. One was a published comparison: the pipeline reproduces a PandaX-4T limit to a ratio of 1.0.
3. Compared the brief against the source and found that I had overestimated how much infrastructure was already in place. The brief asked it to extend a CMB likelihood term that the code did not yet have, pointed at the wrong version block for one of GAMBIT's direct-detection backends, and assumed a GAMBIT model would work with a model family it does not support.
4. Stopped, as the brief requires, and wrote:

> The brief says to stop for confirmation if the source files contradict it, and some do, so I have not started Stage 1. Apart from creating that file, nothing has changed in any repo.

5. Listed five decisions it needed from me, each with a proposed answer.

**Why it matters.** Nothing was built on a false premise. The cheapest moment to catch a wrong assumption is before the first line of code, and the brief made stopping the default.

## 3. My answer: the physics judgement the agent did not have

I accepted some proposals, tightened others and overruled one. From my reply:

> **CMB term.** Agreed, one-sided. Trace P_ANN_LIMIT to Planck 2018 and F_EFF to its source table in comments, and confirm P_ANN_LIMIT is the 95% one-sided value. Model it as a half-Gaussian in p_ann centred on zero, with sigma set so the 95% one-sided point matches the limit. Use the same limit-to-likelihood convention as the dwarf term.
>
> **Dwarf term.** […] Required test: at the halo best-fit mass, [the code] must reproduce the halo paper's [dwarf comparison]. If it does not, the grid limit is not the one the paper used. Stop and report.
>
> **Local DM density, EFT preset. Do not use 0.42.** The EFT convention (v0 = 238, vesc = 544, year-averaged lab speed around 254) is the standard direct-detection convention LZ reports in (Baxter et al., arXiv:2105.00599), which fixes rho0 = 0.3 GeV/cm^3. The EFT script compares against LZ's published limits, so it implicitly inherits 0.3. Verify all of these values from 2105.00599 and cite them.
>
> **numpy:** do not monkeypatch numpy globally. Add one helper […] and change the call sites to use it.

**What this shows.** The agent proposed a local dark matter density of 0.42 GeV/cm³ because that is what the existing code used. That would have been a silent convention error: a direct-detection comparison against LZ's published limits has to use the convention those limits assume, 0.3 GeV/cm³. Errors like this do not crash anything. They move limits. Catching them is the reviewer's job, and it is the job the engine's human gates are designed around.

I also asked for a test that reproduces a published result before the new dwarf term could be trusted, with an instruction to stop if it failed, and for every constant to cite its source.

Later in the same session I chose a simpler dwarf treatment for now and asked for that to be recorded as a choice, not hidden:

> Option (a) but record that this is a chosen simplification for now (there are dwarf limits using spheroidal profiles which I would rather use but I don't have them on disk yet) […]

The session then hardened the Python pipeline (Stage 1), built GAMBIT-light (Stage 2) and built full GAMBIT, with its shipped scalar-singlet example running end to end (Stage 3), stopping for review at each gate.

## 4. A gate fails, and the threshold stays

In the second session the agent built Fermi-LAT dwarf likelihood tables from GAMBIT for the inelastic model. The first gate checked that interpolating the tables reproduces GAMBIT's own likelihood to within 0.05 in ln L.

**What Claude did, step by step.**

1. Ran the check and reported the failure plainly: "G1 failed as written." The worst point was off by 0.45 against the 0.05 threshold.
2. Diagnosed the cause instead of patching the symptom. Where the old and new grids share points, the two GAMBIT runs agree, so the likelihood itself was right. The error came from straight-line interpolation on a grid too coarse for a likelihood that is steep and curved in the cross section.
3. Tested an obvious alternative (interpolating in a different variable), found it did not help, and proposed the real fix with its cost: a three-times-denser grid.
4. Changed nothing on its own: "I haven't changed the grid or the threshold." It gave me the options and stated that "either way the original failure stays on record."

My reply:

> rerun with a denser sigmav grid, then go on to Package 2

The agent then wrote:

> Rerunning both channels on a 41 x 121 grid (0.05 dex in sigmav). The G1 threshold stays at 0.05, and the failed first result stays on record.

**Outcome.** On the denser grid every point passed, with the worst error at 0.031. The commit message records that the coarser grid failed the gate. Later in the session a second check failed narrowly (0.064 against 0.05). It was again reported, not absorbed, and I again chose to fix the grid rather than move the threshold.

## 5. What this demonstrates

- **I set the rules, conventions and pass criteria before any code**, and the agent works inside them.
- **The agent checks before it builds.** It reproduces existing results, including a published one, before changing anything.
- **Contradictions stop the work.** When my brief was wrong about the code, the agent said so and waited.
- **Failures are reported with a diagnosis and kept on record.** In these sessions no threshold was moved to fit a result.
- **Physics judgement stays with the physicist.** The decisions that change limits, such as which density convention a comparison inherits, are mine.

The pipeline built in these sessions now passes 350 automated tests. Its inelastic kinematics agree with an independent implementation to 0.1%. The complete session logs are kept.
