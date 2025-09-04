<div align="center">
<h1>Databricks Labs — Free Edition</h1>
<p>SQL • Delta Lake • Bronze/Silver/Gold • Versionado com GitHub Repos</p>
<p><em>Workspace: Databricks Free Edition (sem cluster manual, compute inicia ao executar células)</em></p>
</div>

<section>
<h2>Objetivo</h2>
<ul>
  <li>Praticar Databricks (Free Edition) com foco em Spark SQL e Delta Lake.</li>
  <li>Construir notebooks curtos e reprodutíveis (1h, 4x por semana).</li>
  <li>Versionar tudo via GitHub (Databricks Repos).</li>
</ul>
</section>

<section>
<h2>Pré‑requisitos</h2>
<ol>
  <li>Conta no GitHub e um repositório vazio (ex.: <code>databricks-labs-mcunhash</code>).</li>
  <li>Personal Access Token no GitHub (PAT) com escopo <code>repo</code>.</li>
  <li>Conta no Databricks Free Edition e acesso ao Workspace.</li>
</ol>
</section>

<section>
<h2>Conectar Databricks ↔ GitHub (Repos)</h2>
<ol>
  <li>No Databricks: <strong>User Settings &gt; Git integration</strong> → Provider: GitHub → cole o seu <em>PAT</em> → Save.</li>
  <li>Sidebar: <strong>Repos &gt; Add Repo &gt; Clone remote Git repo</strong> → URL HTTPS do seu repo (ex.: <code>https://github.com/mcunhash/databricks-labs-mcunhash.git</code>) → Branch <code>main</code> → Create.</li>
  <li>Crie pastas dentro de <code>/Repos/&lt;seu-usuário&gt;/databricks-labs-mcunhash</code>:
    <pre><code>notebooks/
00_setup
01_sql_basics
02_delta_basics
03_bronze_silver_gold
04_perf_e_bi
README.md</code></pre>
  </li>
</ol>
</section>

<section>
<h2>Setup inicial (Notebook: <code>notebooks/00_setup</code>)</h2>
<p>Execute as células SQL abaixo para criar e usar um database de trabalho.</p>
<pre><code class="language-sql">%sql
CREATE DATABASE IF NOT EXISTS mc_labs;
USE mc_labs;</code></pre>
<p>Teste fontes de dados (use a que estiver disponível no seu Free):</p>
<pre><code class="language-sql">%sql
-- Opção A: catálogo de samples (se disponível)
SELECT * FROM samples.nyctaxi.trips LIMIT 10;

-- Opção B: arquivos de datasets (se expostos no seu workspace)
-- SELECT * FROM csv.`dbfs:/databricks-datasets/nyctaxi/tripdata/yellow_tripdata_2019-01.csv` LIMIT 10;</code></pre>
</section>

<section>
<h2>Semana 1 — SQL básico (Notebook: <code>01_sql_basics</code>)</h2>
<p>Crie uma tabela Delta gerenciada a partir de CSV (ajuste o caminho disponível):</p>
<pre><code class="language-sql">%sql
CREATE OR REPLACE TABLE trips_raw USING delta AS
SELECT * FROM csv.`dbfs:/databricks-datasets/nyctaxi/tripdata/yellow_tripdata_2019-01.csv`;</code></pre>
<p>Consultas de prática:</p>
<pre><code class="language-sql">%sql
-- Agregações
SELECT passenger_count, COUNT(*) trips
FROM trips_raw
GROUP BY passenger_count
ORDER BY trips DESC;

-- CTE + deduplicação
WITH ranked AS (
SELECT *, ROW_NUMBER() OVER (PARTITION BY vendor_id, tpep_pickup_datetime ORDER BY tpep_dropoff_datetime DESC) rn
FROM trips_raw
)
SELECT * FROM ranked WHERE rn = 1;</code></pre>
</section>

<section>
<h2>Semana 2 — Delta Lake (Notebook: <code>02_delta_basics</code>)</h2>
<pre><code class="language-sql">%sql
-- Histórico e propriedades
DESCRIBE HISTORY trips_raw;

-- Simular upsert com MERGE (lote 2 fictício)
CREATE OR REPLACE TABLE trips_bronze AS SELECT * FROM trips_raw;
CREATE OR REPLACE TEMP VIEW trips_lote2 AS
SELECT *, total_amount + 1 AS total_amount FROM trips_raw LIMIT 1000;

MERGE INTO trips_bronze t
USING trips_lote2 s
ON t.vendor_id = s.vendor_id AND t.tpep_pickup_datetime = s.tpep_pickup_datetime
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;

-- Time travel
SELECT COUNT(*) FROM trips_bronze VERSION AS OF 0;</code></pre>
</section>

<section>
<h2>Commits (Git) pelo Databricks</h2>
<ol>
  <li>Abrir o notebook alterado → botão <strong>Git</strong> (topo) → selecione mudanças → mensagem → <strong>Commit &amp; Push</strong>.</li>
  <li>Para trazer mudanças do GitHub: <strong>Pull</strong>.</li>
</ol>
</section>

<section>
<h2>Limitações do Free Edition</h2>
<ul>
  <li>Sem criação de cluster manual; use o compute que inicia automaticamente ao rodar células.</li>
  <li>Recursos como SQL Warehouses, Workflows/Jobs completos e Unity Catalog avançado podem não estar disponíveis.</li>
</ul>
</section>

<section>
<h2>Boas práticas</h2>
<ul>
  <li>Exporte seus notebooks periodicamente: <em>File &gt; Export &gt; Source</em>.</li>
  <li>Commits pequenos e frequentes, com mensagens claras.</li>
  <li>Não coloque dados sensíveis no repositório.</li>
</ul>
</section>
