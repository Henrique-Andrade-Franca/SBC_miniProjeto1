# Diagnóstico de Plantas Domésticas

<a href="https://colab.research.google.com/github/Henrique-Andrade-Franca/SBC_miniProjeto1/blob/main/mini_projeto_1.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

**Disciplina:** Sistemas Baseados em Conhecimento (SBC)  
**Autor:** Henrique de Andrade França

## Descrição do Domínio
Este projeto implementa um Sistema Baseado em Conhecimento focado em **Botânica Básica e Jardinagem Doméstica**. O objetivo do sistema é atuar como um assistente especialista capaz de diagnosticar o problema de uma planta e prescrever a intervenção correta. O motor de inferência avalia os sintomas visuais da planta (estado das folhas e humidade do solo) e o seu ambiente (exposição solar) para deduzir o seu estado de saúde e gerar uma recomendação precisa.

O sistema foi desenvolvido em Python utilizando a biblioteca `experta`, demonstrando os conceitos: encadeamento progressivo (*forward chaining*) estruturado em múltiplos níveis e estratégias de resolução de conflitos com recurso a *Salience* e aos operadores `NOT` e `OR`.

---

## Base de Regras (Linguagem Natural)
O sistema possui 14 regras organizadas em 3 níveis lógicos de encadeamento.

**Regras de Exceção (Resolução de Conflitos):**
* **R1 (Planta Morta):** SE o aspeto geral da planta é declarado como "morta", ENTÃO o sistema recomenda o descarte imediato e bloqueia qualquer tentativa de diagnóstico (*Salience = 100*).

**Nível 1 (Avaliação de Sintomas -> Estado da Planta):**
*(Estas regras só são ativadas SE a R1 não tiver disparado).*
* **R2 (Falta de Água):** SE o solo está seco e as folhas estão murchas, ENTÃO conclui-se que o estado da planta é "desidratada".
* **R3 (Excesso de Água):** SE o solo está encharcado e as folhas estão amarelas, ENTÃO conclui-se que o estado é "excesso de água".
* **R4 (Presença de Pragas):** SE as folhas estão furadas OU comidas, ENTÃO conclui-se que o estado é "ataque de pragas".
* **R5 (Planta Saudável):** SE o solo está húmido e as folhas verdes, ENTÃO conclui-se que o estado é "saudável".

**Nível 2 (Estado da Planta + Ambiente -> Diagnóstico):**
* **R6 (Queimadura Solar):** SE a planta está desidratada E o ambiente é de sol direto, ENTÃO o diagnóstico é estresse térmico/queimadura.
* **R7 (Esquecimento de Rega):** SE a planta está desidratada E o ambiente é de sombra, ENTÃO o diagnóstico é apenas um esquecimento prolongado de rega.
* **R8 (Apodrecimento da Raiz):** SE o estado é de excesso de água, ENTÃO o diagnóstico é o apodrecimento da raiz.
* **R9 (Infestação):** SE o estado é de ataque de pragas, ENTÃO o diagnóstico é uma infestação por insetos mastigadores (lagartas).

**Nível 3 (Diagnóstico -> Recomendação de Ação Final):**
* **R10 (Ação para Queimadura):** SE o diagnóstico for estresse térmico, ENTÃO prescreve regar por imersão e mover para a meia-sombra.
* **R11 (Ação para Esquecimento):** SE o diagnóstico for esquecimento de rega, ENTÃO prescreve rega abundante e a criação de um alarme semanal.
* **R12 (Ação para Raiz Podre):** SE o diagnóstico for apodrecimento da raiz, ENTÃO prescreve o replantio de emergência (cortar raízes e trocar a terra).
* **R13 (Ação contra Pragas):** SE o diagnóstico for infestação por lagartas, ENTÃO prescreve a remoção manual e a aplicação de óleo de Neem.

**Regra Final:**
* **R14 (Impressão):** SE foi gerada uma recomendação final, ENTÃO o sistema imprime a prescrição formatada no ecrã para o utilizador.

---

## Casos de Teste e Rastreio (*Trace*)

### Caso de Teste 1: Encadeamento Profundo (Esquecimento na Sombra)
* **Cenário:** O utilizador informa que a planta está viva, fica na sombra, mas o solo está seco e as folhas murchas.
* **Factos Inseridos:** `Planta(aspecto_geral="viva", ambiente="sombra")`, `Sintoma(solo="seco", folhas="murchas")`
* **Regras Disparadas em Cadeia:** `R2` (Gera: Estado desidratada) -> `R7` (Gera: Diagnóstico de esquecimento) -> `R11` (Gera: Recomendação de rega) -> `R14`.
* **Resultado Esperado:** O motor atravessa os 3 níveis lógicos e sugere rega abundante com alarme.

### Caso de Teste 2: Resolução de Conflitos (Planta Morta)
* **Cenário:** O utilizador submete uma planta que já está completamente seca e sem vida (morta), mas tenta fornecer sintomas de humidade e folhas.
* **Factos Inseridos:** `Planta(aspecto_geral="morta", ambiente="sol_direto")`, `Sintoma(solo="seco", folhas="murchas")`
* **Regras Disparadas em Cadeia:** `R1` -> `R14`.
* **Resultado Esperado:** O diagnóstico é interrompido de imediato. Como `R1` possui a prioridade máxima (*Salience = 100*), ela é disparada primeiro, e o operador `NOT(Recomendacao())` nas regras de nível 1 impede que o sistema tente "salvar" ou diagnosticar uma planta irrecuperável.

### Caso de Teste 3: Excesso de Cuidado (Afogando a planta)
* **Cenário:** O utilizador tem a planta na sombra, mas regou demasiado: a terra está como lama e as folhas começaram a amarelar.
* **Factos Inseridos:** `Planta(aspecto_geral="viva", ambiente="sombra")`, `Sintoma(solo="encharcado", folhas="amarelas")`
* **Regras Disparadas em Cadeia:** `R3` (Gera: Excesso de água) -> `R8` (Gera: Diagnóstico de apodrecimento) -> `R12` (Gera: Recomendação de replantio) -> `R14`.
* **Resultado Esperado:** O motor identifica que a junção de terra encharcada com folhas amarelas exige uma intervenção cirúrgica de replantio para salvar as raízes.
