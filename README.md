# CUPED: Redução de Variância em Experimentos com Randomização por Grupos

<p align="justify">Este projeto simula um experimento A/B com influência entre usuários e compara três abordagens: teste A/B individual, randomização por grupos e randomização por grupos combinada com CUPED. O objetivo é estudar como o ajuste CUPED pode reduzir a dispersão dos dados e ajudar a obter estimativas mais estáveis do efeito do tratamento.</p>

## 1. Configuração da Simulação

<p align="justify">A simulação considera 10.000 usuários distribuídos em 300 grupos. O efeito verdadeiro do tratamento é definido como 5, enquanto a influência entre usuários (efeito de propagação) é definida como 3. Esses valores são conhecidos porque foram configurados diretamente no experimento simulado.</p>

```python
n_users = 10_000
n_clusters = 300
true_effect = 5
spillover_effect = 3
```

<p align="justify">O efeito verdadeiro igual a 5 funciona como referência para avaliar se os métodos conseguem estimar adequadamente o impacto do tratamento. O efeito de propagação representa a influência que usuários tratados podem exercer sobre outros usuários, o que pode distorcer um teste A/B quando a randomização é feita individualmente.</p>

## 2. Experimento A/B Individual e Influência entre Usuários

<p align="justify">No teste A/B individual, os usuários são distribuídos entre controle e tratamento. Quando existe influência entre usuários, a comparação pode misturar o efeito direto da intervenção com o efeito de propagação. Na execução registrada, o método ingênuo estimou um efeito de 6,14, acima do efeito verdadeiro de 5, indicando viés nessa simulação.</p>

## 3. Randomização por Grupos

<p align="justify">Na randomização por grupos, os usuários de um mesmo grupo recebem a mesma condição experimental. Essa estratégia ajuda a lidar com a influência entre pessoas do mesmo grupo. Na simulação, o método estimou um efeito de 4,88, próximo do valor verdadeiro de 5.</p>

## 4. Redução de Variância com CUPED

<p align="justify">O CUPED utiliza uma variável auxiliar relacionada à métrica de interesse para ajustar os resultados. Quando essa variável ajuda a explicar diferenças entre usuários, o ajuste pode reduzir a dispersão dos valores observados.</p>

```python
# Forma geral do ajuste CUPED:
# Y ajustado = Y observado - theta * (X - média de X)
Y_cuped = Y - theta * (X - X_mean)
```

<p align="justify">Na execução registrada, a combinação de randomização por grupos com CUPED estimou um efeito de 4,93, também próximo do valor verdadeiro de 5. A variância calculada dos valores individuais da métrica passou de 358,8 para 104,1, uma redução de aproximadamente 71%.</p>

## 5. Resultados da Simulação

| Abordagem | Efeito estimado | Variância dos valores individuais da métrica |
|---|---:|---:|
| A/B individual ingênuo | 6,14 | — |
| Randomização por grupos | 4,88 | 358,8 |
| Randomização por grupos + CUPED | 4,93 | 104,1 |

<p align="justify">Os resultados mostram que, nesta simulação, o A/B individual produziu uma estimativa acima do efeito verdadeiro, enquanto os métodos com randomização por grupos ficaram próximos de 5. O CUPED também reduziu a variância dos valores individuais da métrica de 358,8 para 104,1, o que indica menor dispersão desses valores após o ajuste.</p>

## 6. Como Interpretar a Redução de 71%

<p align="justify">É importante distinguir duas medidas. A variância dos valores individuais indica o quanto os dados dos usuários estão espalhados em torno da média. Já a variância ou o erro-padrão da estimativa do efeito indica o quanto o resultado do experimento poderia variar se o teste fosse repetido com novas amostras.</p>

<p align="justify">Portanto, a redução de aproximadamente 71% observada neste código se refere à variância dos valores individuais da métrica. Ela não significa, por si só, que a incerteza da estimativa do efeito também caiu 71%. Para demonstrar diretamente esse ganho de precisão, seria necessário calcular a variância ou o erro-padrão da diferença entre os grupos, respeitando a estrutura de randomização por grupos, e idealmente comparar intervalos de confiança.</p>

<p align="justify">Em termos simples: o código mostra que, depois do CUPED, os valores individuais ficaram menos dispersos. Ainda falta medir diretamente quanto isso tornou mais preciso o resultado final do experimento. Essa ressalva não invalida a simulação; apenas delimita corretamente o que os números demonstram.</p>

## 7. Gráfico da Redução de Variância

<p align="justify">Insira abaixo o gráfico produzido pelo notebook. Para que ele apareça no GitHub, mantenha o arquivo de imagem na mesma pasta deste relatório ou ajuste o caminho conforme a estrutura do repositório.</p>

<p align="center">
<img src="https://github.com/rodfloripa/Projeto99/blob/main/cuped.png" alt="Comparação da distribuição da métrica antes e depois do CUPED" width="800">
</p>

## 8. Conclusão

<p align="justify">Nesta simulação, a randomização por grupos produziu uma estimativa de efeito próxima do valor verdadeiro, reduzindo o problema de viés associado à influência entre usuários. A aplicação do CUPED diminuiu em aproximadamente 71% a variância calculada dos valores individuais da métrica e manteve a estimativa do efeito próxima de 5: o resultado foi 4,93.</p>

<p align="justify">A conclusão correta é que o CUPED reduziu a dispersão da métrica nesta simulação. Para afirmar quantitativamente que houve uma melhora equivalente na precisão da estimativa do efeito, o próximo passo é calcular e comparar os erros-padrão ou intervalos de confiança dos efeitos estimados, considerando a randomização por grupos.</p>

</p>

