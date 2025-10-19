# CASOS DE TESTE - SUB1_SchemaQuality

## OBJETIVO
Validar funcionamento do SUB1 em cenários reais e edge cases.

---

## TESTE 1: Identificação de Fonte (Cenário Normal)

**INPUT:**
- arquivo_nome = "YSMM_VI_ACOMP_202401.xlsx"
- Arquivo contém dados SAP padrão

**PASSOS:**
1. Executar BLOCO 01
2. Verificar mensagem: "Fonte identificada: SAP_YSMM_VI_ACOMP"

**ESPERADO:**
- fonte = "SAP_YSMM_VI_ACOMP"
- status_bloco = "OK"

**VALIDAR:**
- ✅ fonte != "DESCONHECIDO"
- ✅ Mensagem clara impressa
- ✅ Aguardou validação usuário

---

## TESTE 2: Mapeamento via Dicionário (Cenário Normal)

**INPUT:**
- DataFrame com colunas: ["Centro", "Sigla", "Cód Grupo de produto", "Expedição c/ Veículo"]
- Dicionário DE-PARA carregado com estes campos

**PASSOS:**
1. Executar BLOCO 02 (Carregar Dicionário)
2. Executar BLOCO 03 (Mapear campos)
3. Verificar log_decisoes

**ESPERADO:**
- mapeamento["Centro"] = "Centro" (método: DICIONARIO, confiança: 1.0)
- mapeamento["Sigla"] = "Sigla" (método: DICIONARIO, confiança: 1.0)
- Total mapeados via dicionário: 4

**VALIDAR:**
- ✅ Todos 4 campos encontrados no dicionário
- ✅ Confiança = 1.0 para todos
- ✅ Método = "DICIONARIO"
- ✅ lista_nao_mapeados = []

---

## TESTE 3: Detecção Automática (Cenário Normal)

**INPUT:**
- DataFrame com coluna: "Expedicao Veiculo" (sem acento, sem "c/")
- Campo NÃO está no dicionário

**PASSOS:**
1. Executar BLOCO 04 (Detecção Automática)
2. Verificar se detectou "expedicao" → "Expedição c/ Veículo"

**ESPERADO:**
- mapeamento["Expedicao Veiculo"] = "Expedição c/ Veículo"
- método: "DETECCAO_AUTO"
- confiança: >= 0.7 (match parcial)

**VALIDAR:**
- ✅ Campo detectado corretamente
- ✅ Confiança >= 0.5
- ✅ Mensagem: "DETECCAO: 'Expedicao Veiculo' → 'Expedição c/ Veículo' (confianca=0.7)"

---

## TESTE 4: Conversão de Tipos (Cenário Normal)

**INPUT:**
- DataFrame com:
  - "Centro": tipo object, valores ["5025", "5030", "5035"]
  - "Expedição c/ Veículo": tipo object, valores ["1.500.000,00", "2.000.000,50"]

**PASSOS:**
1. Executar BLOCO 05 (Validar e Converter Tipos)
2. Verificar tipos após conversão

**ESPERADO:**
- "Centro": convertido para int64 → [5025, 5030, 5035]
- "Expedição c/ Veículo": convertido para float64 → [1500000.0, 2000000.5]
- total_nans = 0 (nenhum valor inválido)

**VALIDAR:**
- ✅ df_temp["Centro"].dtype == int64
- ✅ df_temp["Expedição c/ Veículo"].dtype == float64
- ✅ Nenhum NaN gerado
- ✅ Mensagem: "CONVERTIDO: 'Centro' de object para int64"

---

## TESTE 5: Criação de Flags (Cenário Normal)

**INPUT:**
- DataFrame validado com 100 registros
- 5 registros com % VI extremo (outliers)
- 3 registros com VI impossível (maior que expedição)
- 2 registros com expedição = 0

**PASSOS:**
1. Executar BLOCO 06 (Criar Flags de Qualidade)
2. Contar registros flagados

**ESPERADO:**
- Flag_Outlier_% VI: 5 registros (5%)
- Flag_VI_Impossivel: 3 registros (3%)
- Flag_Expedicao_Zero: 2 registros (2%)
- Flag_Dados_Suspeitos: 10 registros (10%) - soma sem duplicar

**VALIDAR:**
- ✅ 7 colunas de flags criadas
- ✅ Todas flags são tipo bool
- ✅ Flag_Dados_Suspeitos é TRUE para qualquer outra flag TRUE
- ✅ Mensagens claras: "Flag criada: 5 outliers em '% VI' (5.00%)"

---

## TESTE 6: Edge Case - Campo Obrigatório Ausente

**INPUT:**
- DataFrame SEM coluna "Centro" (campo obrigatório)

**PASSOS:**
1. Executar BLOCO 08 (Preparar Saída)
2. Validar campos críticos

**ESPERADO:**
- status = "ERRO_CAMPOS_AUSENTES"
- Mensagem: "ERRO: Campo obrigatório ausente: 'Centro'"
- Processo INTERROMPIDO

**VALIDAR:**
- ✅ status != "OK"
- ✅ Erro claro e específico
- ✅ Lista de campos presentes vs esperados mostrada
- ✅ NÃO prosseguiu para próximos blocos

---

## TESTE 7: Edge Case - Conversão Impossível

**INPUT:**
- DataFrame com "Centro" contendo valores: ["5025", "ABC", "5030"]

**PASSOS:**
1. Executar BLOCO 05 (Conversão)
2. Tentar converter "Centro" para int64

**ESPERADO:**
- Conversão com errors='coerce'
- "ABC" → NaN
- total_nans = 1
- Mensagem: "ATENCAO: 1 valores inválidos convertidos para NaN"

**VALIDAR:**
- ✅ Conversão não falhou (usou coerce)
- ✅ NaNs registrados
- ✅ Avisos claros
- ✅ status_bloco = "OK" (com aviso, não erro)

---

## TESTE 8: Edge Case - DataFrame Vazio

**INPUT:**
- df_bruto com 0 registros (apenas cabeçalhos)

**PASSOS:**
1. Tentar executar qualquer bloco

**ESPERADO:**
- status = "ERRO_DADOS_VAZIOS"
- Mensagem: "ERRO: DataFrame está vazio (0 registros)"

**VALIDAR:**
- ✅ Erro detectado logo no início
- ✅ Processo não avançou
- ✅ Mensagem clara

---

## TESTE 9: Atualização Dicionário (Cenário Normal)

**INPUT:**
- log_decisoes com:
  - ("Expedicao Veiculo", "Expedição c/ Veículo", "DETECCAO_AUTO", 0.95)
  - ("Variacao Interna", "Variação Interna", "DETECCAO_AUTO", 0.90)

**PASSOS:**
1. Executar BLOCO 07 (Atualizar Dicionário)
2. Verificar JSON salvo

**ESPERADO:**
- Dicionário atualizado com 2 novos campos
- Confiança >= 0.90 para ambos
- Arquivo salvo: Dicionario_DePara_Template.json

**VALIDAR:**
- ✅ JSON válido após salvamento
- ✅ Campos adicionados em fonte correta
- ✅ Metadados inclusos (data_mapeamento, metodo_deteccao)
- ✅ Mensagem: "DICIONARIO ATUALIZADO: 'Expedicao Veiculo' → ..."

---

## TESTE 10: Integração Completa (End-to-End)

**INPUT:**
- Arquivo real: YSMM_VI_ACOMP_202401.xlsx
- Dicionário parcialmente preenchido

**PASSOS:**
1. Executar TODOS os blocos (01-08)
2. Validar cada etapa
3. Obter resultado final

**ESPERADO:**
- status = "OK"
- df_validado com tipos corretos
- Flags criadas
- Dicionário atualizado
- metadados completo

**VALIDAR:**
- ✅ Nenhum erro bloqueante
- ✅ Todos campos críticos presentes
- ✅ Log_decisoes completo
- ✅ Estatísticas flags corretas
- ✅ Pronto para enviar ao Orquestrador

---

## RESUMO DE VALIDAÇÕES

| Teste | Cenário | Resultado Esperado | Prioridade |
|-------|---------|-------------------|------------|
| 1 | Fonte normal | Identificação OK | ALTA |
| 2 | Mapeamento normal | Dicionário funciona | ALTA |
| 3 | Detecção automática | Sinonimos detectados | ALTA |
| 4 | Conversão tipos | int64/float64 OK | CRÍTICA |
| 5 | Flags qualidade | 7 flags criadas | CRÍTICA |
| 6 | Campo ausente | Erro claro | CRÍTICA |
| 7 | Conversão falha | Coerce + aviso | MÉDIA |
| 8 | DataFrame vazio | Erro imediato | ALTA |
| 9 | Atualização dict | JSON salvo OK | MÉDIA |
| 10 | End-to-End | Tudo integrado | CRÍTICA |

---

**Total de Testes**: 10 (6 normais + 4 edge cases)
**Cobertura**: Todos os 8 blocos + 4 cenários erro
**Tempo estimado**: ~30 min (com arquivo real)