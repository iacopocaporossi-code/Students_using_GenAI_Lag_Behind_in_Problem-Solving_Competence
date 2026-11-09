Agent-based model of problem-solving competence (PSC) in a high-school physics classroom, where students solve exercises alone, with peers, 
or with the support of Generative AI (GenAI).

This repository contains the simulation code for the paper:

Students using GenAI lag behind in problem-solving competence: an agent-based study of classroom networks  I. Caporossi, C. Källner, K. Levanaitė, G. Lorenzini,
M. Re Fiorentin, V. Socci, I. Stanzani, M. Baratto Complexity72h – Complexity Next Gen, Northeastern University London, 22–26 June 2026

**Model overview**

The model simulates a classroom of N = 30 students (agents) connected through a peer network. Over T = 60 time steps students face one physics exercise per step.
Each student i has three state variables:
- $C_i^t$ which indicates the	Problem-solving competence	and it is initialized as $\mathcal{N}(195.9,\ 1.288)$ (INVALSI 2025)
- $\alpha_i^t$	which indicates the	GenAI dependence, in [0, 1], initially fixed as	0.5
- $\tau_i^t$	which indicates the reliance on peers, in [0, 1], initially fixed as	0.5

At each time step every student goes through five stages:

Strategy choice. The student uses GenAI with probability $P(s_i^t=1) = \sigma\big(w_C(k_m^t + k_p^t + ps^t - \Delta C_i^t) + w_\alpha \alpha_i^t - w_\tau \tau_i^t\big)$, 
                    with $w_C = w_\alpha = w_\tau = 0.33$.
Peer interaction. A student who does not use GenAI cooperates with a randomly selected network neighbour with probability $\tau_i^t$. 
                    If the neighbour is more competent, the student's competence is drawn uniformly between the two competences.
Exercise result. Without GenAI the exercise is solved with probability 0.54 (INVALSI 2025).
Competence and GenAI-dependence update.
No GenAI and exercise solved: the student gains $k_m^t + k_p^t + ps^t$ and $\alpha$ decreases.
No GenAI and exercise not solved: no gain from the exercise; $\alpha$ unchanged.
GenAI: the student gains $U(0, k_{m,AI}^t) + U(0, k_{p,AI}^t)$ minus the CPSS term $ps_{AI}^t$, and $\alpha$ increases.
Reliance-on-peers update. For students who cooperated, $\tau$ increases if their competence is above its initial value and decreases if it is below.

Only stages 1 and 2 are choices of the agent; stages 3–5 are outcomes and state updates.

**Notebook structure**

A. Import libraries

Imports the packages used in the notebook: agentpy, networkx, random, numpy, pandas, math, matplotlib, scikit-learn and scipy.

B. Data and functions useful for the model

Sigmoid function, used in the strategy choice.
INVALSI statistics. Computes the mean and standard deviation of the INVALSI 2025 achievement levels (1–5) from their percentages (INVALSI report, p. 92).
Exercise dataset. Loads Exercises_abilities.csv (semicolon-separated) and sorts it by Exercise. Required columns:
Exercise (Exercise identifier), Maths_knowledge, Physics_knowledge, Problem_solving	(Rubric scores of the textbook solution), Total	(it is the sum of the three human scores), 
AI_Maths_knowledge, AI_Physics_knowledge, AI_Problem_solving	(Rubric scores of the GenAI solution) and AI_total	(it is the sum of the three GenAI scores).
The exercises come from M. Virdis, Esercizi risolti di fisica. They were scored with the rubric described in Section 2 of the paper; the GenAI solutions were produced with GPT-5.5 Instant (May 2026).
Normalisation. Total is min-max normalised to [0, 0.5]. Every other column is multiplied by the same per-exercise factor, so each normalised total is still the sum of its components.

C. Model

C.0 Setting the background. Turns the normalised exercise components into lists (maths_knowledge, physics_knowledge, problem_solving and their _AI counterparts). The model reads them at each time step.
C.1 Student agent (Student, ap.Agent). Defines the state variables and the five behaviours, one per stage of the dynamics:
Choose_strategy	--> Use or not use GenAI
Interact_with_friends	--> Cooperate with a random neighbour or work alone
Exercises_results	--> Exercise solved or not
Update_competencies	--> Competence and GenAI-dependence update
Update_reliance_on_friends	--> Reliance-on-peers update
C.2 Classroom model (Classroom, ap.Model). Creates the students and the peer network, places each student on a node, runs the five stages in order (step) and records the fractions of students using and not using GenAI (update). 
Available networks, all with N = 30 and expected mean degree ⟨k⟩ = 4:
C.3a One run. Defines the parameters and runs a single classroom.
C.3b Multiple runs. Runs 1000 independent classrooms with the same parameters (ap.Experiment). Each iteration has new initial competences and a new network realisation.
C.3c Competence trajectories. Plots the competence of every student over time.
C.3d Final competences. Collects the competence of every student at the last time step.
The remaining cells of section C show the agent-level table and the range of additional_competence, plot the histogram of final competences and save the agent-level time series to a CSV (e.g. students_output_SBM_modified_noAI.csv).
D. Analysis

Accesses the results of the single run (results.variables.Classroom, results.variables.Student) and of the multiple runs (results_multiple.variables.Student). 
Plots the trajectories of competence, GenAI dependence and reliance on peers for each student of the single run.

Requirements
- Python ≥ 3.9
- agentpy
- networkx
- numpy
- pandas
- scipy
- scikit-learn
- matplotlib
