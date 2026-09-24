---
name: sonarqube
description: Use when analyzing code quality, identifying technical debt, fixing vulnerabilities, and ensuring adherence to SonarQube quality gates.
---

# Skill: Validação de Qualidade (SonarQube)

Seu objetivo é garantir que o código seja limpo, seguro e testável, atuando como um revisor rigoroso antes de qualquer merge.

## 1. Tratamento de Code Smells e Bugs
- Ao detectar ou ser notificado de um Code Smell, não apenas silencie a ferramenta. Refatore o código seguindo os princípios SOLID e DRY.
- Elimine complexidade ciclomática desnecessária (ifs aninhados, métodos gigantes).
- Substitua códigos duplicados por funções utilitárias compartilhadas ou herança apropriada.

## 2. Segurança e Vulnerabilidades
- Nunca adicione chaves de API, senhas ou tokens hardcoded no código. Use variáveis de ambiente.
- Previna injeções (SQL, XSS, etc) sanitizando entradas e utilizando ORMs ou Prepared Statements nas camadas de dados.

## 3. Cobertura de Testes (Quality Gates)
- Usar as condições efetivamente configuradas para **código novo** (cobertura, bugs, vulnerabilidades, duplicação) e os testes pertinentes à alteração; não declarar aprovado só porque a análise começou.
- Ao adicionar regras de negócio, cobrir comportamentos e caminhos de erro com testes úteis. Se houver falha de cobertura, verificar quais caminhos realmente faltam.

## 4. PR, check obrigatório e revisão

- Quando integrado ao repositório, disparar scanner em cada PR relevante e publicar o resultado do quality gate como check. Resultado reprovado ou inconclusivo não passa; configurar o check como obrigatório no ruleset/branch protection quando exigido pela entrega.
- Scanner efêmero em CI não implica servidor SonarQube efêmero: confirmar URL, credenciais e retenção do servidor persistente antes de configurar o workflow. Nunca executar análise de código de fork com segredo acessível a scripts não confiáveis.
- Se o gate falhar, registrar evidências no PR e corrigir findings de código novo; mover a issue para `Request changes` apenas se houver vínculo e permissão e isso corresponder à automação do Project. Esse Status não equivale a uma review `REQUEST_CHANGES` nem bloqueia merge sem check obrigatório.
