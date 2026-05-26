# Two-Phase Offloading and Resource Allocation Heuristic

This repository contains the pseudo-code for the heuristic admission control and adaptive bidirectional offloading algorithm proposed in our SBrT paper regarding TN-NTN coexistence.

## Algorithm Overview

**Require:** * Set of UEs: $\mathcal{U}$
* Total RBs: $R_{TN}$, $R_{NTN}$
* SNR vectors: $\gamma_{TN}$, $\gamma_{NTN}$
* Offloading Method: $M$

**Initialize:** * $\mathcal{U}_{TN} \gets \emptyset$, $\mathcal{U}_{NTN} \gets \emptyset$, $\mathcal{U}_{out} \gets \emptyset$
* $\gamma_{th} \gets -5$ dB *(Physical outage threshold)*

---

### Phase 1.1: Physical Cut and Initial Association
**For all** $u \in \mathcal{U}$ **do:**
* **If** $\gamma_{TN}[u] < \gamma_{th}$ **and** $\gamma_{NTN}[u] < \gamma_{th}$ **then**
  * $\mathcal{U}_{out} \gets \mathcal{U}_{out} \cup \{u\}$
* **Else if** $\gamma_{TN}[u] \ge \gamma_{NTN}[u]$ **then**
  * $\mathcal{U}_{TN} \gets \mathcal{U}_{TN} \cup \{u\}$
* **Else**
  * $\mathcal{U}_{NTN} \gets \mathcal{U}_{NTN} \cup \{u\}$
* **End if**

### Phase 1.2: Adaptive Bidirectional Offloading
* $O_{TN}, O_{NTN} \gets \text{MeasureInitialOutage}(\mathcal{U}_{TN}, \mathcal{U}_{NTN})$
* **If** $O_{TN} \ge O_{NTN}$ **then**
  * $\text{Passes} \gets [(\text{TN} \rightarrow \text{NTN}), (\text{NTN} \rightarrow \text{TN})]$
* **Else**
  * $\text{Passes} \gets [(\text{NTN} \rightarrow \text{TN}), (\text{TN} \rightarrow \text{NTN})]$
* **End if**

**For each** $pass \in \text{Passes}$ **do:**
* $src, dst \gets \text{Origin and Destination networks of } pass$
* $\mathcal{G} \gets \{u \in \mathcal{U}_{src} \mid \gamma_{dst}[u] \ge \gamma_{th}\}$ *(Identify eligible Givers)*
* $\mathcal{G}_{sorted} \gets \text{FilterAndSort}(\mathcal{G}, M)$ *(Apply Method M logic)*

* **For all** $u \in \mathcal{G}_{sorted}$ **do:**
  * $\text{Temporary move } u \text{ from } src \text{ to } dst$
  * $\text{Evaluate global metrics (Outage and Median Capacity)}$
  * **If** move improves global metrics (no outage increase & respects fairness) **then**
    * $\text{Confirm move}$
  * **Else**
    * $\text{Revert move}$
  * **End if**
* **End for**

### Phase 2: Resource Allocation
* $\mathbf{C}_{TN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{TN}, R_{TN}, \gamma_{TN})$
* $\mathbf{C}_{NTN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{NTN}, R_{NTN}, \gamma_{NTN})$

**Ensure:** Final Sets $\mathcal{U}_{TN}, \mathcal{U}_{NTN}, \mathcal{U}_{out}$ and Capacity Vectors $\mathbf{C}_{TN}, \mathbf{C}_{NTN}$.

```latex
\begin{algorithm}[htbp]
\caption{Two-Phase Adaptive Bidirectional Offloading and Resource Allocation}
\label{alg:offloading}
\begin{algorithmic}[1]
\Require Set of UEs $\mathcal{U}$, Total RBs $R_{TN}, R_{NTN}$, SNR vectors $\gamma_{TN}, \gamma_{NTN}$, Method $M$
\State \textbf{Initialize:} $\mathcal{U}_{TN} \gets \emptyset$, $\mathcal{U}_{NTN} \gets \emptyset$, $\mathcal{U}_{out} \gets \emptyset$
\State $\gamma_{th} \gets -5$\,dB \Comment{Physical outage threshold}

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

\Statex \textbf{\% Phase 1.2: Adaptive Bidirectional Offloading}
\State $O_{TN}, O_{NTN} \gets \text{MeasureInitialOutage}(\mathcal{U}_{TN}, \mathcal{U}_{NTN})$
\If{$O_{TN} \geq O_{NTN}$}
    \State $\text{Passes} \gets [(\text{TN} \rightarrow \text{NTN}), (\text{NTN} \rightarrow \text{TN})]$
\Else
    \State $\text{Passes} \gets [(\text{NTN} \rightarrow \text{TN}), (\text{TN} \rightarrow \text{NTN})]$
\EndIf

\ForAll{$pass \in \text{Passes}$}
    \State $src, dst \gets \text{Origin and Destination networks of } pass$
    \State $\mathcal{G} \gets \{u \in \mathcal{U}_{src} \mid \gamma_{dst}[u] \geq \gamma_{th}\}$ \Comment{Eligible Givers}
    \State $\mathcal{G}_{sorted} \gets \text{FilterAndSort}(\mathcal{G}, M)$ \Comment{Apply Method M logic}
    
    \ForAll{$u \in \mathcal{G}_{sorted}$}
        \State $\text{Temporarily move } u \text{ from } src \text{ to } dst$
        \State $\text{Evaluate global Outage and Median Capacity}$
        \If{move degrades global metrics}
            \State $\text{Revert move}$ \Comment{Anti-thrashing property}
        \EndIf
    \EndFor
\EndFor

\Statex \textbf{\% Phase 2: Resource Allocation}
\State $\mathbf{C}_{TN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{TN}, R_{TN}, \gamma_{TN})$
\State $\mathbf{C}_{NTN} \gets \text{RoundRobinScheduler}(\mathcal{U}_{NTN}, R_{NTN}, \gamma_{NTN})$

\Ensure Final Sets $\mathcal{U}_{TN}, \mathcal{U}_{NTN}, \mathcal{U}_{out}$ and Capacity Vectors $\mathbf{C}_{TN}, \mathbf{C}_{NTN}$
\end{algorithmic}
\end{algorithm}
