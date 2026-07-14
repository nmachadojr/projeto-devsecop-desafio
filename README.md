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
Primeiramente, professor Pedro, tive certas dificuldades com o ambiente do GitHub e entender exatamente
o que fazer de alteração na pipeline para gerar os erros de vulnerabilidade e finalmente gerar a URL de
produção. Procurei buscar tirar minhas dúvidas com a IA do ChatGPT e ela me orientou no que eu tinha de
dificuldade para conseguir efetuar o deploy completo do pipeline sem erros.

O passo inicial foi implementar os steps de segurança da pipeline incompletos. No arquivo pipeline.yml fiz
as seguintes inserções no código:

      # ❌ PASSO 3: IMPLEMENTE AQUI — Secrets Scanning com Gitleaks
      # Dica: use a action oficial "gitleaks/gitleaks-action@v2"
      - name: 🔑 Secrets Scanning
        uses: gitleaks/gitleaks-action@v2
        env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  
        # SUBSTITUA o run acima pela implementação correta com "uses:"

      # ❌ PASSO 4: IMPLEMENTE AQUI — SAST com Semgrep
      # Dica: instale com "pip install semgrep" e rode "semgrep scan --config auto --error src/"
      - name: 🔍 SAST - Semgrep
        run: |
            python3 -m pip install semgrep
            semgrep scan --config auto --error src/
        # SUBSTITUA pelo comando real: pip install semgrep && semgrep scan --config auto --error src/

      # ❌ PASSO 5: IMPLEMENTE AQUI — SCA com Grype
      # Dica: instale com "curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b 
      /usr/local/bin"
      # e rode "grype dir:. --fail-on medium"
      - name: 📦 SCA - Grype
        uses: anchore/scan-action@v6
        with:
            path: "."
            fail-build: true
            severity-cutoff: medium  
        # SUBSTITUA pelo comando real de instalação + execução do Grype

Após isso, utilizei as ferramentas de segurança automatizadas (Gitleaks, Semgrep e Grype) para detectar as
vulnerabilidades propositais no código e corrigi-las.

Segue abaixo a função de cada ferramenta com a identificação e correção das vulnerabilidades:

GITLEAKS
É uma ferramenta de segurança de código aberto projetada para identificar e prevenir o vazamento de dados 
sensíveis (como senhas, chaves de API e tokens) deixados acidentalmente em repositórios Git. Ela vasculha 
tanto os arquivos atuais quanto todo o histórico de commits procurando por padrões de risco.

Na primeira execução do Actions o Gitleaks no Secrets Scanning acusou o seguinte:

Warning: 🛑 Leaks detected, see job summary for details

O segredo identificado no código foi esse:

const API_KEY = ""; (removi o token daqui, pois ao tentar fazer deploy definitivo em produção, o Gitleaks
acusou vazamento de dado sensível no README.md)

Porém, como falado pelo professor Pedro na aula de 13/07, a linha:

const DB_PASSWORD = ""; (fiz a mesma coisa aqui removendo a senha)

também é considerada um segredo. Logo, ambas foram removidas, pois se trata de uma vulnerabilidade
senhas ou quaisquer outras informações sigilosas virem hardcoded.

Salvei o arquivo script.js sem essas linhas e fiz novo commit e push pro repositório.
Executei o Actions novamente e o Gitleaks passou sem acusar vulnerabilidades.

SEMGREP
É uma ferramenta de análise estática de código (SAST) de código aberto e focada em segurança, projetada para encontrar bugs, vulnerabilidades e falhas de lógica de negócios. Ele analisa o código-fonte rapidamente sem precisar executá-lo. 

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
É uma ferramenta open-source gratuita que funciona como um scanner de vulnerabilidades para software. Ela foi projetada para identificar falhas de segurança conhecidas (CVEs) em ambientes de desenvolvimento e produção.

O Grype passou sem acusar vulnerabilidades.

Depois de todos os acertos no código script.js indicados , o pipeline.yml foi devidamente implantado em 
produção.

## URL de Produção
https://nmachadojr.github.io/projeto-devsecop-desafio/

