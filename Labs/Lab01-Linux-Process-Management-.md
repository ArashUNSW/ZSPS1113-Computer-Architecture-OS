\begin{tikzpicture}[
  font=\sffamily\scriptsize,
  >=Latex,
  box/.style={draw=black!65, rounded corners=2pt, align=center,
              inner sep=4pt, minimum height=10mm, fill=white},
  inputbox/.style={box, fill=blue!5, draw=blue!55!black},
  core/.style={box, fill=green!6, draw=green!48!black},
  decisionbox/.style={box, fill=orange!7, draw=orange!70!black},
  artifact/.style={box, fill=violet!5, draw=violet!55!black},
  meta/.style={box, fill=gray!7, draw=gray!60, dashed},
  flow/.style={-{Latex[length=2.1mm,width=1.45mm]}, line width=.75pt, draw=black!82},
  aux/.style={-{Latex[length=1.9mm,width=1.3mm]}, dashed, line width=.65pt, draw=gray!68},
  boundary/.style={draw=black!45, densely dashed, rounded corners=4pt, inner sep=7pt},
  lab/.style={font=\sffamily\tiny, text=black!68, align=center, fill=white, inner sep=1pt}
]

% Inputs
\node[inputbox, text width=40mm] (event) at (-62mm,10mm) {\textbf{Governed operation $x_i$}\\[-1pt]
service, actor/role, purpose,\\stage, action, data scope};

\node[inputbox, text width=40mm] (evidence) at (-62mm,-10mm) {\textbf{Authoritative evidence $b_i$}\\[-1pt]
identity, consent/approval, audit,\\provenance, destination, registry,\\emergency state};

\node[meta, text width=40mm] (qmeta) at (-62mm,-34mm) {\textbf{Predictive metadata $q_i$}\\[-1pt]
score / model output\\[-1pt]\emph{record or display only}};

% Trusted mediation plane
\node[core, text width=32mm] (adapter) at (-17mm,0mm) {\textbf{Trusted evidence adapter}\\[-1pt]
authenticates and normalizes\\security-sensitive attributes};

\node[core, text width=38mm] (catalog) at (28mm,20mm) {\textbf{Versioned policy catalog $\Pi^v$}\\[-1pt]
pinned, immutable enforcement contract};

\node[core, text width=40mm] (worker) at (28mm,0mm) {\textbf{Stateless EGM policy worker}\\[-1pt]
policy predicates + emergency-state resolution\\[-1pt]
$\rightarrow$ strongest graded outcome};

\node[core, text width=40mm] (writer) at (28mm,-28mm) {\textbf{Decision artifact writer}\\[-1pt]
event ID + policy version +\\reasons + obligations + evidence refs};

% Outputs
\node[decisionbox, text width=40mm] (gate) at (78mm,0mm) {\textbf{Action gate / response}\\[-1pt]
\texttt{allow} $\mid$ \texttt{warn} $\mid$ \texttt{escalate} $\mid$ \texttt{block}\\[-1pt]
enforce current operation};

\node[artifact, text width=40mm] (record) at (78mm,-28mm) {\textbf{Decision-linked record}\\[-1pt]
policy IDs + evidence refs\\reasons + audit/provenance};

\node[meta, text width=40mm] (crosswalk) at (78mm,-47mm) {\textbf{Optional standards crosswalk}\\[-1pt]
policy ID $\rightarrow$ framework category\\[-1pt]\emph{reporting aid only}};

% Explicit connection points on adapter west edge
\coordinate (adapterInTop) at ($(adapter.west)+(0,4mm)$);
\coordinate (adapterInBot) at ($(adapter.west)+(0,-4mm)$);

% Safe bend coordinates strictly left of adapter
\coordinate (bendTop) at (-37mm,10mm);
\coordinate (bendBot) at (-37mm,-10mm);

% Input arrows: rightward into adapter, no reversal
\draw[flow] (event.east) -- (bendTop) |- (adapterInTop);
\draw[flow] (evidence.east) -- (bendBot) |- (adapterInBot);

% Main path
\draw[flow] (adapter.east) -- (worker.west);
\draw[flow] (catalog.south) -- (worker.north);
\draw[flow] (worker.east) -- (gate.west);
\draw[flow] (worker.south) -- (writer.north);
\draw[flow] (writer.east) -- (record.west);
\draw[aux] (record.south) -- (crosswalk.north);

% Predictive metadata path only to writer, routed below trust plane
\coordinate (metaBend1) at (-35mm,-34mm);
\coordinate (metaBend2) at (-35mm,-43mm);
\coordinate (writerUnder) at ($(writer.south)+(0,-7mm)$);
\draw[aux] (qmeta.east) -- (metaBend1) -- (metaBend2) -- (writerUnder) -- (writer.south);
\node[lab] at (-10mm,-43mm) {metadata only};

% Trusted boundary
\begin{scope}[on background layer]
  \node[boundary, fit=(adapter)(catalog)(worker)(writer)] (trust) {};
\end{scope}
\node[font=\sffamily\tiny\bfseries, text=black!67, anchor=north west,
      fill=white, inner sep=1pt]
  at ($(trust.north west)+(2mm,-2mm)$) {Trusted mediation plane};

\end{tikzpicture}
