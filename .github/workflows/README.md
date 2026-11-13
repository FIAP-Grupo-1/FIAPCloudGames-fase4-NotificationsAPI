# GitHub Actions Workflows

## CI/CD Pipeline (ci.yml)

Executa automaticamente em push e pull requests.

### Etapas:
1. **Checkout** - Faz checkout do código
2. **Setup .NET** - Configura .NET 10.0
3. **Restore** - Restaura dependências do projeto
4. **Build** - Compila a solução em modo Release
5. **Tests** - Executa todos os testes unitários
6. **Code Style** - Valida regras de código (Roslyn)
7. **Check Changes** - Verifica se há mudanças não commitadas

### Triggers:
- Push em `main` (CI + deploy da Lambda)
- Pull requests para `main` ou branches `feature/**`

## Artifacts

Os workflows armazenam automaticamente:
- `test-results` - Resultados dos testes em formato TRX

## Status Badge

Adicione ao README principal:

```markdown
![CI/CD Pipeline](https://github.com/joao-malvetoni-alta-horizon/FIAPCloudGames-fase2-NotificationsAPI/workflows/CI%2FCD%20Pipeline/badge.svg)
```

## CD: deploy da Lambda (job `deploy` do ci.yml)

Depois de um push na `main` (merge de PR), com o build e os testes verdes, o job publica as duas
funções com o SAM na stack `fcg-notifications-serverless`:

1. **Credenciais AWS por OIDC**: o GitHub assume a role `fcg-gha-notifications`. Nenhuma access key
   fica guardada no GitHub.
2. **`sam build`** e **`sam deploy`** (parâmetros no `samconfig.toml`). Quem aplica o change set é a
   role de serviço `fcg-cfn-notifications`, limitada aos recursos `fcg-notifications*`.
3. O `AutoPublishAlias: prod` do template publica uma versão nova e move o alias `prod` para ela: as
   filas SQS passam a entregar para o código novo sem parar de consumir.
4. O resumo da execução mostra a versão do alias `prod` de cada função.

As duas roles ficam no repositório Orchestration (`infra/cloudformation/github-oidc.yaml`,
`make oidc-deploy`). Pull requests rodam só o CI, sem deploy.
