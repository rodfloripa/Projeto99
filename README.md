

# CUPED: Redução de Variância em Experimentos com Randomização por Grupos

<p align="justify">Este projeto simula um experimento A/B com influência entre usuários e compara três abordagens: teste A/B individual, randomização por grupos e randomização por grupos combinada com CUPED. A versão atual do código também calcula uma estimativa aproximada do erro-padrão do efeito, considerando os grupos de randomização, para avaliar não apenas a dispersão dos dados individuais, mas também a incerteza do resultado do experimento.</p>

## 1. Configuração da Simulação

<p align="justify">A simulação utiliza 10.000 usuários distribuídos em 300 grupos. O efeito verdadeiro do tratamento é definido como 5, e o efeito de propagação entre usuários é definido como 3. Como esses valores foram configurados diretamente na simulação, eles servem como referência para avaliar as estimativas produzidas pelos métodos.</p>

```python
N_USERS = 10000
N_CLUSTERS = 300
TRUE_EFFECT = 5
SPILLOVER_EFFECT = 3
```

<p align="justify">A variável `X_pre` representa uma métrica anterior à intervenção, neste caso descrita como tempo de uso do aplicativo nos 14 dias anteriores. Ela é usada pelo CUPED como variável auxiliar para explicar parte das diferenças entre os usuários.</p>

## 2. Experimento A/B Individual e Influência entre Usuários

<p align="justify">No teste A/B individual, os usuários são distribuídos separadamente entre controle e tratamento. O código simula uma situação em que o resultado de cada pessoa também depende da proporção de pessoas tratadas em seu grupo. Essa influência entre usuários viola a hipótese de que o resultado de uma pessoa não é afetado pelo tratamento recebido por outras pessoas.</p>

<p align="justify">Na execução registrada, o teste A/B individual estimou um efeito de 6,14, acima do efeito verdadeiro de 5. Esse resultado ilustra como a influência entre usuários pode distorcer a estimativa do efeito direto do tratamento.</p>

## 3. Randomização por Grupos

<p align="justify">Na randomização por grupos, todos os usuários de um mesmo grupo recebem a mesma condição experimental. Isso ajuda a evitar a mistura entre usuários tratados e não tratados dentro do mesmo grupo, que é a fonte de influência simulada no primeiro desenho.</p>

<p align="justify">O método de randomização por grupos estimou um efeito de 4,88, próximo do valor verdadeiro de 5. O código também calcula um erro-padrão aproximado usando a variação das médias dos grupos tratados e de controle, em vez de tratar todos os usuários como observações independentes.</p>

## 4. Ajuste CUPED

<p align="justify">O CUPED usa uma variável anterior à intervenção, relacionada à métrica de interesse, para ajustar os resultados. A ideia é retirar parte da variação associada a diferenças que já existiam antes do experimento, mantendo a comparação entre os grupos.</p>

```python
# Ajuste CUPED:
# Y ajustado = Y observado - theta * (X - média de X)
df["Y_cuped"] = df["Y_cluster"] - theta * (df["X_pre"] - X_mean)
```

<p align="justify">No código, o coeficiente `theta` é calculado a partir da covariância entre a métrica e a variável auxiliar no grupo de controle, dividida pela variância da variável auxiliar. Depois, o ajuste é aplicado à métrica dos usuários. A versão atual avalia tanto a variância dos valores individuais quanto o erro-padrão aproximado da estimativa do efeito.</p>

## 5. Como o Erro-Padrão é Calculado

<p align="justify">Como a atribuição do tratamento ocorre por grupo, o código agrega a métrica de cada grupo e calcula a variação entre as médias dos grupos tratados e dos grupos de controle. O erro-padrão é estimado a partir dessas variações e do número de grupos em cada condição experimental.</p>

```python
se = np.sqrt(
    treat_clusters.var() / n_t +
    control_clusters.var() / n_c
)
```

<p align="justify">Esse cálculo é uma aproximação baseada nas médias dos grupos. Ele é mais coerente com a unidade de randomização do que calcular o erro-padrão tratando todos os usuários como independentes. Ainda assim, para uma análise aplicada rigorosa, convém verificar o estimador e o método de inferência adequados ao desenho experimental, especialmente se os tamanhos dos grupos forem diferentes.</p>

## 6. Resultados da Simulação

| Abordagem | Efeito estimado | Erro-padrão aproximado | Intervalo de confiança de 95% |
|---|---:|---:|---:|
| A/B individual ingênuo | 6,14 | 0,423 | Não calculado no código para esse método |
| Randomização por grupos | 4,88 | 0,386 | [4,12; 5,64] |
| Randomização por grupos + CUPED | 4,93 | 0,208 | [4,52; 5,34] |

<p align="justify">O efeito verdadeiro é 5. As estimativas dos dois métodos com randomização por grupos ficaram próximas desse valor. Após o CUPED, o efeito estimado foi 4,93, e o erro-padrão aproximado caiu de 0,386 para 0,208. Os intervalos de confiança apresentados foram calculados pelo código usando a estimativa do efeito mais ou menos 1,96 vezes o erro-padrão.</p>

## 7. Interpretação das Reduções

| Indicador | Resultado |
|---|---:|
| Redução da variância dos valores individuais da métrica | 71,0% |
| Redução da variância estimada do efeito (erro-padrão ao quadrado) | 71,1% |
| Aumento relativo da precisão, definida como 1 / erro-padrão² | 245,5% |
| Redução equivalente do tamanho amostral necessário para alcançar o mesmo erro-padrão | 71,1% |

<p align="justify">A versão atual do código permite uma conclusão mais forte do que a versão anterior: além de reduzir a variância dos valores individuais em 71,0%, o CUPED reduziu em aproximadamente 71,1% a variância estimada do efeito, calculada como o erro-padrão ao quadrado. O erro-padrão caiu de 0,386 para 0,208, indicando que, nesta simulação, a estimativa ajustada ficou menos incerta.</p>

<p align="justify">O aumento relativo de precisão de 245,5% usa a definição adotada no código: precisão igual ao inverso da variância do estimador, ou seja, 1 / SE². Isso significa que a precisão calculada dessa forma ficou cerca de 3,45 vezes maior. Não significa que o efeito estimado aumentou 245,5%; o efeito estimado continuou próximo de 5.</p>

<p align="justify">A redução equivalente de tamanho amostral de 71,1% é uma equivalência matemática baseada na razão entre as variâncias estimadas, sob condições comparáveis. Não significa que o experimento real possa necessariamente usar exatamente 71,1% menos usuários sem outras verificações.</p>

## 8. Gráfico da Redução de Variância

<p align="justify">O notebook gera um histograma que compara os valores da métrica no grupo de controle antes e depois do ajuste CUPED. Para que a imagem apareça no GitHub, mantenha o arquivo `cuped_variance_reduction.png` na mesma pasta deste relatório ou ajuste o caminho conforme a estrutura do repositório.</p>

<p align="center">
<img src="https://github.com/rodfloripa/Projeto99/blob/main/cuped.png" alt="Comparação da distribuição da métrica antes e depois do CUPED" width="800">
</p>

## 9. Conclusão

<p align="justify">Nesta simulação, o teste A/B individual estimou um efeito de 6,14, acima do valor verdadeiro de 5, em razão da influência entre usuários incorporada ao cenário. A randomização por grupos produziu uma estimativa de 4,88, e a combinação com CUPED produziu uma estimativa de 4,93.</p>

<p align="justify">O CUPED reduziu a variância dos valores individuais da métrica em 71,0% e a variância estimada do efeito em aproximadamente 71,1%. O erro-padrão aproximado caiu de 0,386 para 0,208, enquanto o intervalo de confiança de 95% passou de [4,12; 5,64] para [4,52; 5,34]. Assim, os resultados desta simulação indicam uma estimativa mais precisa após o ajuste.Na prática, reduzir a variância em 71% permite detectar o efeito de uma mudança com mais confiança, usando a mesma quantidade de dados. Isso é muito útil em experimentos A/B, testes de funcionalidades e avaliação de modelos.</p>

<p align="justify">Esses números devem ser interpretados como resultados de uma simulação e de um cálculo aproximado do erro-padrão no nível dos grupos. Antes de aplicar o procedimento em um experimento real, é recomendável validar o método de inferência e considerar possíveis diferenças no tamanho dos grupos. A conclusão sustentada pelo código é que, neste cenário simulado, o CUPED reduziu a dispersão dos dados e também a incerteza estimada do efeito.</p>
