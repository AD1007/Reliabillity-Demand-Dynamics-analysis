\section*{Wikimedia Traffic Reliability \& Demand Dynamics Engine}

\noindent
\textbf{Technology Stack} \\
Python 3.8+ \\
DuckDB (Out-of-Core OLAP Engine) \\
Dataset Size: 41GB Raw Telemetry Logs \\
System Status: Production Ready

\bigskip

\section*{Project Overview}

This repository contains a production-grade data pipeline and volatility analysis engine designed to evaluate measurement reliability across large-scale consumer traffic derived from Wikimedia Foundation telemetry.

The primary objective is to quantify system stability, detect structural demand shifts, and mathematically isolate statistical noise from actionable business growth. Rather than relying on pre-processed datasets, the system ingests, structures, and analyzes 41GB of raw machine-generated server logs representing approximately 15 billion monthly page views.

The project intentionally simulates a realistic enterprise telemetry environment in which analysts must build reliable measurement frameworks directly on top of raw infrastructure data.

\bigskip

\section*{System Architecture}

Processing highly granular compressed telemetry logs on standard hardware (16GB RAM) requires strict out-of-core design principles. The architecture avoids full-memory loading and instead leverages an embedded OLAP query engine.

\bigskip

\textbf{Pipeline Flow}

\begin{enumerate}
\item Raw Telemetry Ingestion (742 compressed .gz files)
\item Streaming Query Execution through DuckDB
\item Canonical Aggregation Layer
\begin{itemize}
\item Hourly Project-Level Views
\item Global Monthly Metrics
\end{itemize}
\item Artifact Serialization to Columnar Storage (Parquet)
\item Volatility Modeling and Noise Estimation
\item Anomaly Detection Backtesting using Rolling Statistical Thresholds
\end{enumerate}

\bigskip

\section*{Operational Pipeline Design}

The system is organized into four major operational stages.

\subsection*{1. Out-of-Core Ingestion}

The pipeline uses DuckDB to directly query compressed \texttt{.gz} telemetry logs via streaming execution. This eliminates the need for full file extraction and bypasses memory limitations typically associated with Pandas-based pipelines.

This design allows tens of gigabytes of log data to be processed on commodity hardware without requiring distributed infrastructure.

\subsection*{2. Dimensionality Reduction}

Raw event-level telemetry is collapsed into structured analytical tables using SQL aggregation. The system produces temporal warehouse tables at hourly and project granularity while preserving immutable raw logs for auditability.

These derived tables allow millisecond-latency analytical queries while maintaining a clean separation between raw and analytical layers.

\subsection*{3. Statistical Modeling}

The engine evaluates volatility using the Coefficient of Variation:

\[
CV = \frac{\sigma}{\mu}
\]

where

\begin{itemize}
\item $\sigma$ represents the standard deviation of traffic measurements
\item $\mu$ represents the mean traffic level
\end{itemize}

This metric provides a normalized measure of variability, enabling comparison across segments with drastically different traffic scales.

\subsection*{4. Automated Artifact Serialization}

Downstream analytical datasets are automatically exported as compressed Parquet artifacts. These include:

\begin{itemize}
\item Concentration Metrics
\item Structural Breadth Indicators
\item Volatility Profiles
\end{itemize}

Columnar storage enables efficient downstream integration with BI platforms and allows reproducible historical auditing.

\bigskip

\section*{Quantitative Findings: The Aggregation Illusion}

The analysis reveals a significant discrepancy between aggregate system stability and segment-level volatility.

\subsection*{Macro-Level Stability}

Global aggregate traffic exhibits controlled temporal variability.

\[
CV_{global} \approx 0.15
\]

At the platform level, traffic appears highly stable when measured through aggregated dashboards.

\subsection*{Micro-Level Instability}

Segment-level traffic exhibits substantially higher volatility.

\[
CV_{project\_average} \approx 1.04
\]

Long-tail segments display extreme burst-driven behavior.

\[
CV_{extreme} > 9.0
\]

This indicates that individual projects can experience dramatic fluctuations even while the platform aggregate appears stable.

\subsection*{Decision Noise Band}

Variance reduction across the 742-hour observation window allows estimation of the natural statistical noise floor for month-over-month measurements.

\[
Noise\ Floor < 1\%
\]

This implies that executive decisions reacting to aggregate growth signals below the 1–2\% range are statistically likely to be responding to random hourly fluctuation rather than genuine structural demand shifts.

\bigskip

\section*{Backtesting and Production Monitoring}

To operationalize these findings, the repository includes a multi-resolution monitoring simulator implemented through the function:

\texttt{simulate\_anomaly\_detection\_backtest}

The monitoring architecture combines:

\begin{itemize}
\item Rolling Z-score anomaly detection
\item Three-sigma statistical thresholding
\item Exponential smoothing for volatility stabilization
\end{itemize}

This framework suppresses false positives from naturally volatile segments while still detecting true structural anomalies.

\bigskip

\textbf{Simulated Monitoring Performance (10,000 Parallel Streams)}

\begin{tabular}{|l|c|}
\hline
Metric & Performance \\
\hline
Detection F1 Score & 0.98 \\
True Positive Rate (Recall) & 0.97 \\
False Positive Rate & 0.0075 \\
Verified Monthly Noise Floor & $<0.85\%$ \\
\hline
\end{tabular}

\bigskip

\section*{Reproducibility and Setup}

\subsection*{Prerequisites}

\begin{itemize}
\item Python 3.8 or higher
\item At least 20GB of available disk space
\end{itemize}

\bigskip

\subsection*{Repository Setup}

\begin{verbatim}
git clone https://github.com/yourusername/wiki-traffic-reliability.git
cd wiki-traffic-reliability

python -m venv venv
source venv/bin/activate

# Windows
venv\Scripts\activate

pip install duckdb pandas numpy matplotlib statsmodels scikit-learn
\end{verbatim}

\bigskip

\subsection*{Data Acquisition}

Download the raw December 2025 telemetry dataset from the Wikimedia Foundation dumps.

\begin{verbatim}
wget -r -np -nH --cut-dirs=4 -A "*.gz" \
https://dumps.wikimedia.org/other/pageviews/2025/2025-12/
\end{verbatim}

Ensure that the download directory matches the path expected by the ingestion scripts.

\bigskip

\subsection*{Pipeline Execution}

Execute the pipeline in three sequential stages.

\begin{verbatim}
# 1. Build the DuckDB warehouse and run canonical aggregations
python build_warehouse.py

# 2. Execute volatility analysis and generate Parquet artifacts
python run_volatility_analysis.py

# 3. Run structural change and anomaly detection backtests
python run_backtests.py
\end{verbatim}
