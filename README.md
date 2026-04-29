# two-phase-offloading-and-resource-allocation-heuristic-algorithm
Pseudo-code for paper regarding TN-NTN vertical offloading

\begin{algorithm}[htbp]
\caption{Two-Phase Offloading and Resource Allocation Heuristic}
\label{alg:offloading}
\begin{algorithmic}[1]
\Require Set of UEs $\mathcal{U}$, Total RBs $R_{TN}, R_{NTN}$, SNR vectors $\mathbf{\gamma}_{TN}, \mathbf{\gamma}_{NTN}$, Method $M$
\State \textbf{Initialize:} $\mathcal{U}_{TN} \gets \emptyset$, $\mathcal{U}_{NTN} \gets \emptyset$, $\mathcal{U}_{out} \gets \emptyset$
\State $\gamma_{th} \gets -5$ dB \Comment{Physical outage threshold}

\Statex \textbf{\% Phase 1.1: Physical Cut and Initial Association}
\ForAll{$u \in \mathcal{U}$}
    \If{$\gamma_{TN}[u] < \gamma_{th}$ \textbf{and} $\gamma_{NTN}[u] < \gamma_{th}$}
        \State $\mathcal{U}_{out} \gets \mathcal{U}_{out} \cup \{u\}$
    \ElsIf{$\gamma_{TN}[u] \geq \gamma_{NTN}[u]$}
        \State $\mathcal{U}_{TN} \gets \mathcal{U}_{TN} \cup \{u\}$
    \Else
        \State $\mathcal{U}_{NTN} \gets \mathcal{U}_{NTN} \cup \{u\}$
    \EndIf
\EndFor

\Statex \textbf{\% Phase 1.2: Vertical Offloading (TN $\rightarrow$ NTN)}
\State $\mathcal{G} \gets \{u \in \mathcal{U}_{TN} \mid \gamma_{NTN}[u] \geq \gamma_{th}\}$ \Comment{Identify eligible Givers}
\State $\mathcal{G}_{filtered} \gets \text{ApplyFilter}(\mathcal{G}, M)$ \Comment{e.g., All Givers or Outage only}
\State $\mathcal{G}_{sorted} \gets \text{Sort}(\mathcal{G}_{filtered}, M)$ \Comment{Sort by SNR or RB Cost based on $M$}

\ForAll{$u \in \mathcal{G}_{sorted}$}
    \If{TN constraint requires offloading \textbf{and} NTN has available capacity}
        \State $\mathcal{U}_{TN} \gets \mathcal{U}_{TN} \setminus \{u\}$
        \State $\mathcal{U}_{NTN} \gets \mathcal{U}_{NTN} \cup \{u\}$
    \EndIf
\EndFor

\Statex \textbf{\% Phase 2: Resource Allocation}
\State $\mathbf{C}_{TN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{TN}, R_{TN}, \mathbf{\gamma}_{TN})$
\State $\mathbf{C}_{NTN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{NTN}, R_{NTN}, \mathbf{\gamma}_{NTN})$

\Ensure Final Sets $\mathcal{U}_{TN}, \mathcal{U}_{NTN}, \mathcal{U}_{out}$ and Capacity Vectors $\mathbf{C}_{TN}, \mathbf{C}_{NTN}$
\end{algorithmic}
\end{algorithm}
