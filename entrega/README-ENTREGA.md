# Correção do relatório de reservas

O `onfly_report` lê reservas de hotel de um CSV e gera um relatório JSON com o resumo, os totais por mês e o ranking de hotéis. O relatório estava saindo com números errados.

## Como os problemas foram encontrados

A investigação começou pelo `pytest`. Os cinco testes falhavam. Um deles, por exemplo, esperava 3 noites para uma reserva de 30/01 a 02/02, mas recebia −28. Isso ajudou a localizar o erro na conta das datas. A leitura do código também mostrou que o programa parava na reserva 200, apesar de o CSV ter 300 registros.

## O que foi corrigido

1. **Leitura do CSV.** O laço parava ao chegar em 200 reservas. Com a remoção desse limite, a leitura vai até o fim do arquivo.
2. **Filtro por status.** O filtro excluía só `CANCELLED`, então `NO_SHOW` entrava nos valores e nas noites. Agora só `CONFIRMED` entra nesses cálculos, e os três status continuam aparecendo na contagem do resumo.
3. **Cálculo das noites.** A conta subtraía apenas o dia do mês, o que dava um resultado errado na virada do mês. A correção usa `(checkout - checkin).days` e multiplica a diferença pelo número de quartos.
4. **Conversão para reais.** O código dividia o valor pela taxa usando `float`. A correção multiplica `amount * rate` com `Decimal` e mantém o arredondamento na hora de gerar o JSON.
5. **Ranking de hotéis.** A ordenação colocava os menores valores primeiro. A chave `(-valor, hotel_id)` coloca os maiores primeiro e mantém o desempate pelo ID em ordem crescente.

## Como validar

Na raiz do projeto, execute no PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe -m pytest -q

$env:PYTHONPATH = "src"
.\.venv\Scripts\python.exe -m onfly_report --input data\exemplo-validacao.csv --rates data\taxas_cambio.csv --output output\exemplo-validacao.json
.\.venv\Scripts\python.exe -m onfly_report --input data\reservas_hoteis.csv --rates data\taxas_cambio.csv --output output\report.json
```

## Resultados

A validação foi feita com Python 3.14.7 e pytest 8.3.4. Antes das correções, os cinco testes falhavam. Depois, o resultado foi `5 passed`.

A **amostra** bateu com o enunciado. Foram 12 reservas, sendo 9 confirmadas, 2 canceladas e 1 no-show. O relatório mostrou 28 noites de quarto, total de R$ 7.566,00 e diária média de R$ 270,21.

O **CSV completo** gerou um relatório com 300 reservas, sendo 235 confirmadas, 31 canceladas e 34 no-show. Foram 1.347 noites de quarto, total de R$ 396.170,00 e diária média de R$ 294,11. Um script separado conferiu as noites, o total e a média.
