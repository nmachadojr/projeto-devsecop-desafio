# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech.
Você receberá este projeto com vulnerabilidades propositais e uma pipeline incompleta.
Seu objetivo é **implementar a pipeline de segurança** e **corrigir as vulnerabilidades**.

## Estado atual
A pipeline está **incompleta**. Os steps de segurança precisam ser implementados por você.

## Sua missão
1. Implementar os steps de segurança no `pipeline.yml`
2. Fazer a pipeline **quebrar** ao detectar os problemas
3. Corrigir as vulnerabilidades encontradas
4. Fazer a pipeline **passar** com tudo verde ✅
5. Documentar o funcionamento da pipeline neste README

## O que implementar
- [ ] Secrets Scanning com **Gitleaks**
- [ ] SAST com **Semgrep**
- [ ] SCA com **Grype**
- [ ] Deploy com **GitHub Pages**

## Como a pipeline funciona
> **Substitua este bloco pela sua explicação após implementar a pipeline.**
GITLEAKS
Na primeira execução do Actions o Gitleaks no Secrets Scanning acusou o seguinte:

Warning: 🛑 Leaks detected, see job summary for details

O segredo identificado no código foi esse:

const API_KEY = "ghp_xK92mNpL34rTvQ87wZaB56cDeFgHiJkL";

Porém, como falado pelo professor Pedro na aula de 13/07, a linha:

const DB_PASSWORD = "admin@prod#2024";

também é considerada um segredo. Logo, ambas foram removidas, pois se trata de uma vulnerabilidade
senhas ou quaisquer outras informações sigilosas virem hardcoded.

Salvei o arquivo script.js sem essas linhas e fiz novo commit e push pro repositório.
Executei o Actions novamente e o Gitleaks passou sem acusar vulnerabilidades.

SEMGREP

Após a segunda execução do Actions o Semgrep identificou isso:

┌────────────────┐
│ 1 Code Finding │
└────────────────┘
                 
    src/script.js
    ❯❱ javascript.browser.security.eval-detected.eval-detected
          ❰❰ Blocking ❱❱
          Detected the use of eval(). eval() can be dangerous if used to evaluate dynamic content. If this 
          content can be input from outside the program, this may be a code injection vulnerability. Ensure
          evaluated content is not definable by external sources.                                          
          Details: https://sg.run/7ope                                                                     
                                                                                                           
           26┆ eval('console.log("Tarefa adicionada: ' + input.value + '")');


A regra acionada foi:

javascript.browser.security.eval-detected.eval-detected

Trata-se de uma regra bloqueante (Blocking). E por que isso se trata de uma vulnerabilidade? Porque
eval() interpreta uma string como código JavaScript. Se porventura alguém vir a alterar essa linha para
usar entrada do usuário, como por exemplo, eval(input.value); um invasor poderia executar código arbitrário 
no navegador. Por isso o Semgrep segue as recomendações da OWASP e bloqueia qualquer uso de eval().

A correção que fiz foi a seguinte: 

Substitui a linha eval('console.log("Tarefa adicionada: ' + input.value + '")');
por console.log("Tarefa adicionada:", input.value);

O resultado será o mesmo, sem usar eval().

Após efetuar a troca de uma linha de código por outra no arquivo script.js, fiz novo commit e push pro
repositório. Executei o Actions novamente e o Semgrep passou sem acusar vulnerabilidades.

GRYPE

O Grype passou sem acusar vulnerabilidades.

## URL de Produção
> Adicione aqui o link do GitHub Pages após o deploy.
