# CUPED: Redução de Variância em Experimentos com Randomização por Grupos

<p align="justify">Este projeto simula um experimento para medir o efeito de uma mudança sobre o engajamento de usuários. O código compara três abordagens: um experimento A/B individual que permite influência entre usuários, a atribuição do tratamento por grupos e a aplicação do CUPED para reduzir a variabilidade dos resultados. A simulação utiliza 10.000 usuários distribuídos em 300 grupos e define um efeito verdadeiro de 5 unidades.</p>

<p align="justify">O objetivo é demonstrar dois problemas importantes na avaliação de experimentos: a influência de usuários tratados sobre outros usuários do mesmo grupo e a variabilidade natural da métrica observada. A randomização por grupos procura evitar a mistura entre tratamento e controle dentro de um mesmo grupo, enquanto o CUPED utiliza uma medida anterior ao experimento para tornar o resultado menos variável.</p>

# 1. Configuração da Simulação

<p align="justify">A simulação começa definindo a quantidade de usuários, o número de grupos e o efeito que se deseja recuperar. A variável <code>X_pre</code> representa o tempo de uso do aplicativo nos 14 dias anteriores ao experimento. Ela funciona como uma medida anterior ao tratamento, útil para explicar parte do comportamento observado depois.</p>

```python
np.random.seed(42)

N_USERS = 10000
N_CLUSTERS = 300
TRUE_EFFECT = 5
SPILLOVER_EFFECT = 3

df = pd.DataFrame({
    "user_id": range(N_USERS),
    "cluster_id": np.random.choice(range(N_CLUSTERS), N_USERS),
    "X_pre": np.random.normal(100, 20, N_USERS)
})
```

<p align="justify">A função <code>np.random.seed(42)</code> fixa a semente aleatória, permitindo reproduzir a simulação. O efeito verdadeiro é definido como 5. Já o efeito de influência entre usuários é definido como 3 e é usado no primeiro desenho para representar a contaminação entre participantes do mesmo grupo.</p>

# 2. Experimento A/B Individual e Influência entre Usuários

<p align="justify">No primeiro desenho, cada usuário recebe aleatoriamente o tratamento ou o controle. Em seguida, o código calcula a proporção de usuários tratados em cada grupo e adiciona essa influência à métrica observada. Assim, o resultado de uma pessoa pode depender não apenas de seu próprio tratamento, mas também da situação dos demais integrantes do grupo.</p>

```python
df["treatment_naive"] = np.random.binomial(1, 0.5, N_USERS)

cluster_treat_pct = (
    df.groupby("cluster_id")["treatment_naive"].mean().to_dict()
)
df["cluster_treat_pct"] = df["cluster_id"].map(cluster_treat_pct)

df["Y_naive"] = (
    df["X_pre"] * 0.8
    + np.random.normal(0, 10, N_USERS)
    + df["treatment_naive"] * TRUE_EFFECT
    + df["cluster_treat_pct"] * SPILLOVER_EFFECT
)
```

<p align="justify">Esse desenho viola a hipótese de que o resultado de um usuário não é afetado pelo tratamento recebido por outros participantes. Na simulação, a estimativa do efeito foi de <b>6,14</b>, acima do efeito verdadeiro de 5. A diferença ilustra como a influência entre usuários pode distorcer a interpretação do experimento.</p>

# 3. Randomização por Grupos

<p align="justify">Na segunda abordagem, o tratamento é sorteado para cada grupo inteiro. Todos os usuários de um mesmo grupo ficam na mesma condição experimental. Isso reduz o risco de misturar usuários tratados e não tratados dentro do mesmo grupo, que é justamente a fonte de influência simulada no primeiro desenho.</p>

```python
cluster_treatment = {
    c: np.random.binomial(1, 0.5)
    for c in range(N_CLUSTERS)
}
df["treatment_cluster"] = df["cluster_id"].map(cluster_treatment)

df["Y_cluster"] = (
    df["X_pre"] * 0.8
    + np.random.normal(0, 10, N_USERS)
    + df["treatment_cluster"] * TRUE_EFFECT
)
```

<p align="justify">Com essa abordagem, o efeito estimado foi de <b>4,88</b>, próximo do valor verdadeiro de 5. O resultado sugere que a randomização por grupos resolveu o viés introduzido pela influência entre participantes nesta simulação. Em aplicações reais, a análise também deve considerar que usuários do mesmo grupo podem ter resultados correlacionados.</p>

# 4. Redução de Variância com CUPED

<p align="justify">O CUPED ajusta a métrica observada usando uma variável anterior ao experimento que esteja relacionada ao resultado. Neste caso, <code>X_pre</code> representa o tempo de uso do aplicativo antes da intervenção. O ajuste procura remover da métrica posterior a parcela associada a essa característica anterior, preservando a possibilidade de estimar o efeito do tratamento com menor variabilidade.</p>

```python
control_mask = df["treatment_cluster"] == 0

cov_yx = np.cov(
    df.loc[control_mask, "Y_cluster"],
    df.loc[control_mask, "X_pre"]
)[0, 1]

var_x = df["X_pre"].var()
theta = cov_yx / var_x

X_mean = df["X_pre"].mean()
df["Y_cuped"] = (
    df["Y_cluster"] - theta * (df["X_pre"] - X_mean)
)
```

<p align="justify">O parâmetro <code>theta</code> determina a intensidade do ajuste. No código, a covariância entre o resultado e a medida anterior é calculada no grupo de controle, enquanto a variância usada no denominador e a média de referência são calculadas sobre todos os usuários. Para uma implementação metodologicamente mais rigorosa, essas escolhas devem ser definidas de forma consistente com o desenho experimental e com o protocolo de análise.</p>

<p align="justify">A ideia central é que, se a medida anterior explica parte da variação do resultado, subtrair sua contribuição pode aproximar os valores de uma faixa mais estreita. O CUPED não substitui a randomização: ele é uma técnica complementar para melhorar a precisão da estimativa.</p>

# 5. Avaliação dos Resultados

<p align="justify">A função <code>evaluate</code> calcula a diferença entre a média do grupo tratado e a média do grupo de controle, além da variância dos valores da métrica. A diferença entre médias é usada como estimativa do efeito do tratamento.</p>

```python
def evaluate(y_col, treat_col):
    treat = df[df[treat_col] == 1][y_col].mean()
    control = df[df[treat_col] == 0][y_col].mean()
    variance = df[y_col].var()
    return treat - control, variance

eff_naive, var_naive = evaluate("Y_naive", "treatment_naive")
eff_cluster, var_cluster = evaluate("Y_cluster", "treatment_cluster")
eff_cuped, var_cuped = evaluate("Y_cuped", "treatment_cluster")

reduction_pct = (1 - var_cuped / var_cluster) * 100
```

<p align="justify">A execução do código produziu os seguintes resultados:</p>

| Abordagem | Efeito estimado | Variância da métrica |
|---|---:|---:|
| Experimento A/B individual com influência entre usuários | 6,14 | — |
| Randomização por grupos | 4,88 | 358,8 |
| Randomização por grupos com CUPED | 4,93 | 104,1 |

<p align="justify">A variância da primeira abordagem não foi apresentada no resultado impresso. Para as duas abordagens por grupos, o cálculo indica redução de variância de aproximadamente <b>71,0%</b> após o ajuste CUPED. A estimativa do efeito também permaneceu próxima do valor verdadeiro: passou de 4,88 para 4,93, diante de um efeito definido de 5.</p>

# 6. Gráfico da Redução de Variância

<p align="justify">O gráfico compara a distribuição da métrica no grupo de controle antes e depois da aplicação do CUPED. A distribuição ajustada aparece mais concentrada, indicando menor dispersão dos valores após o ajuste pela medida anterior ao experimento.</p>

<p align="center">
<img src="https://github.com/rodfloripa/Projeto99/blob/main/cuped.png" alt="Comparação da distribuição da métrica antes e depois do CUPED" width="800">
</p>

<p align="justify"><b>Interpretação:</b> a distribuição azul, correspondente à métrica ajustada, é mais estreita do que a distribuição cinza, correspondente à métrica sem ajuste. Isso é coerente com a redução de variância calculada pelo código. O gráfico, por si só, mostra a distribuição do grupo de controle; a estimativa do efeito deve continuar sendo avaliada pela comparação entre tratamento e controle.</p>

# 7. Conclusão

<p align="justify">A simulação mostra que o experimento A/B individual pode superestimar o efeito quando há influência entre usuários: a estimativa foi de 6,14, apesar de o efeito verdadeiro ser 5. A randomização por grupos produziu uma estimativa de 4,88, mais próxima do valor definido na simulação, pois manteve cada grupo na mesma condição experimental.</p>

<p align="justify">O principal resultado é que <b>a combinação de randomização por grupos com CUPED apresentou o melhor equilíbrio entre estimativa do efeito e variabilidade</b>. Com CUPED, a estimativa passou de 4,88 para 4,93, permanecendo próxima do efeito verdadeiro de 5, enquanto a variância da métrica caiu de 358,8 para 104,1, uma redução aproximada de 71,0%.</p>

<p align="justify">Esses números demonstram o potencial do ajuste, mas devem ser interpretados no contexto da simulação. A variância calculada pelo código é a variância dos valores individuais da métrica, não diretamente a variância ou o erro-padrão da estimativa do efeito entre grupos. Para uma análise experimental completa, recomenda-se calcular intervalos de confiança e erros-padrão que respeitem a randomização por grupos, além de avaliar a robustez dos resultados em diferentes simulações.</p>
