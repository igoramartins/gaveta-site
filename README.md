# Gaveta — App de Controle Financeiro

App iOS de controle financeiro baseado no método de envelopes. Construído com Flutter (Cupertino), SQLite local e arquitetura BLoC.

---

## Sumário

1. [Visão Geral](#visão-geral)
2. [Stack Tecnológica](#stack-tecnológica)
3. [Estrutura de Pastas](#estrutura-de-pastas)
4. [Funcionalidades](#funcionalidades)
5. [Banco de Dados](#banco-de-dados)
6. [Arquitetura e State Management](#arquitetura-e-state-management)
7. [Sistema de Temas](#sistema-de-temas)
8. [Fluxo de Navegação](#fluxo-de-navegação)
9. [Repositórios e Regras de Negócio](#repositórios-e-regras-de-negócio)
10. [Componentes Compartilhados](#componentes-compartilhados)
11. [Como Rodar](#como-rodar)

---

## Visão Geral

Gaveta é um app de orçamento pessoal que segue o **método de envelopes**: cada despesa é alocada previamente em uma "gaveta" (categoria de gasto), e o usuário distribui o dinheiro disponível entre elas antes de gastar. O app é 100% offline — todos os dados ficam no dispositivo.

**Conceitos-chave:**
- **Orçamento (Budget):** Contexto financeiro principal. Um usuário pode ter múltiplos orçamentos.
- **Gaveta (Envelope):** Categoria de gasto dentro de um orçamento (ex: Alimentação, Combustível).
- **Alocação:** Valor reservado para uma gaveta em um determinado mês.
- **Meta (Target):** Objetivo de poupança de uma gaveta, com frequência mensal, anual ou até uma data.
- **Conta:** Conta bancária, cartão de crédito, dinheiro em espécie ou investimento.

---

## Stack Tecnológica

| Camada | Tecnologia |
|--------|-----------|
| UI | Flutter — Cupertino (iOS-first, sem Material) |
| State Management | `flutter_bloc` 8.1.6 |
| Banco de Dados | SQLite (`sqflite` 2.3.3) |
| Internacionalização | `intl` |
| Geração de IDs | `uuid` |
| Linguagem | Dart |
| Plataforma alvo | iOS |

---

## Estrutura de Pastas

```
gaveta/lib/
├── main.dart                          # Entry point
├── app.dart                           # Root widget + startup gate + splash screen
│
├── core/
│   ├── theme/
│   │   └── app_colors.dart            # Paleta de cores e gradientes
│   └── utils/
│       ├── currency_formatter.dart    # Formatação de valores em reais
│       └── date_formatter.dart        # Formatação de datas e month keys
│
├── data/
│   ├── local/
│   │   └── app_database.dart          # Singleton SQLite, schema e migrações
│   ├── models/                        # Modelos de domínio
│   │   ├── account.dart
│   │   ├── transaction.dart
│   │   ├── envelope.dart
│   │   ├── budget.dart
│   │   └── allocation.dart
│   └── repositories/                  # Interfaces + implementações SQLite
│       ├── accounts_repository.dart
│       ├── sqlite_accounts_repository.dart
│       ├── transactions_repository.dart
│       ├── sqlite_transactions_repository.dart
│       ├── envelopes_repository.dart
│       ├── sqlite_envelopes_repository.dart
│       ├── home_repository.dart
│       ├── sqlite_home_repository.dart
│       ├── merchants_repository.dart
│       └── sqlite_merchants_repository.dart
│
├── features/
│   ├── home/                          # Dashboard principal
│   │   ├── home_screen.dart
│   │   └── bloc/
│   ├── envelopes/                     # Gerenciamento de gavetas
│   │   ├── envelopes_screen.dart
│   │   ├── set_target_screen.dart
│   │   ├── edit_budget_screen.dart
│   │   └── bloc/
│   ├── accounts/                      # Gerenciamento de contas
│   │   ├── accounts_screen.dart
│   │   └── bloc/
│   ├── transactions/                  # Registro de transações
│   │   ├── new_transaction_screen.dart
│   │   ├── edit_transaction_sheet.dart
│   │   ├── merchant_picker_screen.dart
│   │   └── envelope_picker_screen.dart
│   ├── onboarding/                    # Configuração inicial
│   │   └── onboarding_screen.dart
│   ├── settings/                      # Preferências do app
│   │   └── settings_screen.dart
│   └── reports/                       # Relatórios e análises
│       └── reports_screen.dart
│
└── shared/
    └── widgets/
        ├── bottom_nav.dart            # MainScaffold — layout raiz + bottom tabs
        ├── app_card.dart              # Card com tema automático
        ├── allocation_banner.dart     # Banner de alocação expandível
        ├── budget_popover.dart        # Seletor de orçamento
        ├── liquid_glass_button.dart   # Botão glassmorphism
        └── menu_popover.dart          # Menu contextual genérico
```

---

## Funcionalidades

### Home (Dashboard)

A tela principal agrega o estado financeiro do mês atual.

**O que exibe:**
- **Para alocar:** saldo disponível para distribuir entre gavetas
- **Gasto no mês:** total de despesas do mês
- **Cards de cartão de crédito:** fatura atual e limite pessoal com indicadores de disponibilidade
- **Resumo mensal:** meta total, falta cobrir, alocado, gasto
- **Prévia do próximo mês:** quanto já está alocado para o mês seguinte
- **Avisos de gaveta estourada**

**Ações disponíveis:**
- Trocar de orçamento (popover top direito)
- Acessar Configurações
- Alocar fundos manualmente (banner expansível)
- Auto-alocar por 4 estratégias: nome A→Z, nome Z→A, valor menor→maior, valor maior→menor

---

### Gavetas (Envelopes)

Gerenciamento de categorias e envelopes por mês.

**Funcionalidades:**
- Navegar entre meses (setas ← →)
- Ver saldo, alocado e gasto por gaveta
- Inserir valor de alocação diretamente no campo de cada gaveta
- Definir meta de poupança (mensal, anual, ou até uma data)
- Criar, renomear e excluir categorias
- Criar, renomear, alterar tipo e excluir gavetas
- Transferir saldo entre gavetas no mesmo mês
- Auto-alocar fundos disponíveis
- Ver aviso modal quando gaveta está estourada

**Tipos de gaveta (`EnvelopeTipo`):**
| Tipo | Label |
|------|-------|
| `alimentacao` | Alimentação |
| `saude` | Saúde |
| `transporte` | Transporte |
| `educacao` | Educação |
| `combustivel` | Combustível |

> Contas do tipo "benefício" (vale-alimentação, vale-transporte) só aceitam transações em gavetas com o tipo compatível.

---

### Contas

Gerenciamento de contas bancárias e cartões.

**Tipos de conta:**
| Tipo | Descrição |
|------|-----------|
| `cash` | Dinheiro em espécie |
| `checking` | Conta corrente |
| `creditCard` | Cartão de crédito |
| `investment` | Investimento |
| `benefit` | Benefício (VA, VT, VR) |

**Campos específicos por tipo:**
- **Cartão de crédito:** dia de fechamento, dia de vencimento, limite pessoal mensal
- **Benefício:** tipos de gaveta permitidos (lista de `EnvelopeTipo`)

**Cálculo de saldo:**
```
saldo = opening_balance + soma(receitas) - soma(despesas)
```
Transações do tipo `installment_number = 0` (cabeçalho de parcelamento) são excluídas do cálculo.

---

### Transações

Registro de entradas e saídas financeiras.

**Campos de uma transação:**
- Valor, tipo (receita/despesa), data
- Conta debitada/creditada
- Gaveta (opcional)
- Estabelecimento (merchant)
- Descrição livre

**Modos de pagamento para cartão de crédito:**

| Modo | Descrição |
|------|-----------|
| À vista | 1 transação; mês de faturamento calculado pelo dia de fechamento |
| Parcelado | Modelo N+1: 1 cabeçalho (installment_number=0) + N registros individuais por mês |
| Recorrente | 12 registros mensais gerados automaticamente |

**Regra de mês de faturamento (cartão de crédito):**
```
se dia_da_compra <= dia_fechamento  →  fatura do mesmo mês
se dia_da_compra >  dia_fechamento  →  fatura do mês seguinte
```

**Modelo N+1 de parcelamento:**
- `installment_number = 0` → registro "cabeçalho" (usado para editar/deletar o grupo inteiro)
- `installment_number = 1..N` → registros individuais (um por mês de faturamento)

---

### Onboarding

Assistente de configuração exibido na primeira abertura do app.

1. **Tela de boas-vindas** — apresentação do conceito
2. **Nome do orçamento** — o usuário nomeia seu primeiro orçamento
3. **Configuração de categorias** — categorias e gavetas pré-sugeridas editáveis

Ao concluir, grava `onboarding_completed = 'true'` em `app_settings` e navega para o app principal.

---

### Configurações

Acesso pelo popover da Home.

- Seleção de tema: Sistema, Claro, Escuro
- Nome e e-mail do usuário
- Exclusão completa de dados (reset)

---

### Relatórios

Tela reservada para analytics (implementação futura).

---

## Banco de Dados

SQLite local via `sqflite`. Versão atual do schema: **8**.

### Tabelas

#### `budgets`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| name | TEXT | Nome do orçamento |

#### `categories`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| budget_id | TEXT FK | Orçamento pai |
| name | TEXT | Nome da categoria |
| sort_order | INTEGER | Ordem de exibição |

#### `envelopes`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| category_id | TEXT FK | Categoria pai |
| name | TEXT | Nome da gaveta |
| sort_order | INTEGER | Ordem de exibição |
| tipo | TEXT | Tipo (opcional): alimentacao, saude, etc. |

#### `allocations`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| envelope_id | TEXT PK | Gaveta |
| month_key | TEXT PK | Formato: "YYYY-MM" |
| amount_cents | INTEGER | Valor alocado em centavos |

> PRIMARY KEY composta: `(envelope_id, month_key)` — garante 1 alocação por gaveta por mês.

#### `envelope_targets`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| envelope_id | TEXT PK | Gaveta |
| target_cents | INTEGER | Valor alvo em centavos |
| frequency | TEXT | "monthly", "yearly", "byDate" |
| deadline | TEXT | ISO 8601 (nullable) |

#### `accounts`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| budget_id | TEXT | Orçamento |
| name | TEXT | Nome da conta |
| type | TEXT | Tipo da conta |
| opening_balance_cents | INTEGER | Saldo inicial |
| closing_day | INTEGER? | Dia fechamento (CC) |
| due_day | INTEGER? | Dia vencimento (CC) |
| personal_limit_cents | INTEGER? | Limite pessoal mensal (CC) |
| allowed_tipos | TEXT? | Tipos permitidos (benefício), separados por vírgula |

#### `transactions`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| account_id | TEXT | Conta |
| envelope_id | TEXT? | Gaveta (opcional) |
| merchant_id | TEXT? | Estabelecimento (opcional) |
| amount_cents | INTEGER | Valor em centavos |
| type | TEXT | "income" ou "expense" |
| description | TEXT? | Descrição livre |
| date | TEXT | Data ISO 8601 |
| billing_month | TEXT? | Mês de faturamento "YYYY-MM" (CC) |
| installment_group | TEXT? | UUID do grupo de parcelas |
| installment_number | INTEGER? | 0=cabeçalho, 1..N=parcela individual |
| total_installments | INTEGER? | Total de parcelas no grupo |

#### `merchants`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | TEXT PK | UUID |
| name | TEXT | Nome do estabelecimento |

#### `app_settings`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| key | TEXT PK | Chave |
| value | TEXT | Valor |

**Chaves utilizadas:**
| Chave | Descrição |
|-------|-----------|
| `onboarding_completed` | "true" após conclusão do onboarding |
| `active_budget_id` | UUID do orçamento ativo |
| `theme` | "system", "light" ou "dark" |

### Histórico de Migrações

| Versão | O que mudou |
|--------|-------------|
| 1 | Schema inicial (budgets, categories, envelopes, allocations, accounts, app_settings) |
| 2 | Tabela `transactions` |
| 3 | Seed do orçamento "Pessoal" com categorias padrão |
| 4 | Tabela `merchants` + campo `merchant_id` em transactions |
| 5 | Campos de cartão de crédito (closing_day, due_day, personal_limit_cents, billing_month, installment_*) |
| 6 | Campo `tipo` em envelopes + campo `allowed_tipos` em accounts |
| 7 | Modelo N+1 de parcelamento (header records) |
| 8 | Remove seed "Pessoal"; reseta onboarding se não houver orçamentos |

---

## Arquitetura e State Management

### Padrão BLoC

Todos os features seguem o padrão **flutter_bloc**:

```
UI → Event → BLoC → Repository → SQLite
               ↓
            State → UI rebuild
```

- **Events** são imutáveis (`const` constructors)
- **States** extendem `Equatable` (igualdade por valor)
- BLoCs recebem repositórios via construtor (inversão de dependência)
- Cada mutação emite `Loading` → executa operação → emite `Loaded` ou `Error`

### BLoCs do App

#### `HomeBloc`
| Evento | Ação |
|--------|------|
| `HomeLoadRequested` | Carrega resumo do mês atual |
| `BudgetSwitchRequested(name)` | Troca orçamento ativo |
| `BudgetCreateRequested(name)` | Cria novo orçamento com template |

| Estado | Descrição |
|--------|-----------|
| `HomeInitial` | Estado inicial |
| `HomeLoading` | Carregando |
| `HomeLoaded(summary, budgetNames)` | Dados carregados |
| `HomeError(message)` | Erro |

#### `EnvelopesBloc`
| Evento | Ação |
|--------|------|
| `EnvelopesLoadRequested` | Carrega mês atual |
| `EnvelopesMonthChanged(month)` | Navega para outro mês |
| `EnvelopeAllocationSaved(id, cents, monthKey)` | Salva alocação |
| `CategoryAdded(name)` | Cria categoria |
| `EnvelopeAdded(categoryId, name, tipo)` | Cria gaveta |
| `EnvelopeTargetSet(id, cents, frequency, deadline)` | Define meta |
| `EnvelopeUpdated(id, name, tipo)` | Edita gaveta |
| `EnvelopeDeleted(id)` | Exclui gaveta |
| `EnvelopeAutoAllocateRequested(cents, order)` | Distribui fundos |
| `EnvelopeTransferRequested(from, to, month, cents)` | Transfere alocação |

#### `AccountsBloc`
| Evento | Ação |
|--------|------|
| `AccountsLoadRequested` | Carrega todas as contas |
| `AccountAdded(...)` | Cria conta |
| `AccountUpdated(...)` | Edita conta |
| `AccountDeleted(id)` | Exclui conta |

Estado especial `AccountDeleteBlocked` é emitido quando a conta possui transações em gavetas (exclusão bloqueada para integridade dos dados).

### Padrão de Repositório

Interfaces abstratas desacoplam a UI do SQLite:

```dart
abstract class AccountsRepository {
  Future<AccountsSummary> getAccountsSummary();
  Future<void> addAccount(...);
  Future<void> updateAccount(...);
  Future<bool> deleteAccount(String accountId);
}

class SqliteAccountsRepository implements AccountsRepository { ... }
```

---

## Sistema de Temas

### Implementação

O tema é controlado por um `ValueNotifier<String>` global em `app.dart`:

```dart
final themeNotifier = ValueNotifier<String>('system');
```

O `CupertinoApp` usa um `builder` para sobrescrever o `platformBrightness` do `MediaQuery` quando o tema não é "system":

```dart
builder: theme == 'system'
    ? null
    : (context, child) => MediaQuery(
          data: MediaQuery.of(context).copyWith(
            platformBrightness: theme == 'dark'
                ? Brightness.dark
                : Brightness.light,
          ),
          child: child!,
        ),
```

Cada widget detecta o tema com:
```dart
final isDark = MediaQuery.platformBrightnessOf(context) == Brightness.dark;
```

### Paleta de Cores (`AppColors`)

| Constante | Hex | Uso |
|-----------|-----|-----|
| `primary` | #0047C0 | Azul primário da marca |
| `accent` | #007AFF | Azul iOS |
| `gradStart` | — | Início do gradiente de botões |
| `ok` | #00E5A0 | Verde (positivo, disponível) |
| `warn` | #FFBB00 | Amarelo (atenção) |
| `bad` | #FF3B5C | Vermelho (estourado, negativo) |
| `bgDark` | #000000 | Fundo dark mode |
| `bgLight` | #DCE8FB | Fundo light mode |

**Cards:**
- Dark: `Color(0xFF1C1C1E)` (padrão iOS dark)
- Light: `CupertinoColors.white`

**Separadores:**
- Dark: `Color(0xFF38383A)`
- Light: `Color(0xFFF0F0F0)`

---

## Fluxo de Navegação

```
main() → GavetaApp
             │
             └─ _StartupGate
                    │
                    ├── [carregando] → _SplashScreen (logo animado)
                    │
                    ├── [onboarding_completed = false] → OnboardingScreen
                    │                                         │
                    │                                   (conclui) → MainScaffold
                    │
                    └── [onboarding_completed = true] → MainScaffold
                                                              │
                                          ┌───────────────────┼───────────────────┐
                                          │                   │                   │
                                       HomeTab         EnvelopesTab         AccountsTab
                                          │
                              ┌───────────┼───────────┐
                              │           │           │
                          Settings   BudgetPicker  NewTransaction (modal)
```

### Modais e Sheets

- `NewTransactionSheet` — bottom sheet (botão central)
- `SetTargetScreen` — push (Editar gaveta → Definir meta)
- `SettingsScreen` — push (popover da Home)
- `OverBudgetSheet` — modal (aviso de gaveta estourada)
- `MerchantPickerScreen` — push dentro do fluxo de transação
- `EnvelopePickerScreen` — push dentro do fluxo de transação

---

## Repositórios e Regras de Negócio

### Cálculo de "Para Alocar"

```
para_alocar = saldo_total_das_contas - total_alocado_no_mes_atual
```

Onde `saldo_total` considera `opening_balance + receitas - despesas` de todas as contas (excluindo cabeçalhos de parcelamento).

### Cálculo de Gasto por Gaveta

Para cartão de crédito:
```sql
WHERE envelope_id = ? AND billing_month = 'YYYY-MM'
  AND (installment_number IS NULL OR installment_number != 0)
```

Para outras contas:
```sql
WHERE envelope_id = ? AND strftime('%Y-%m', date) = 'YYYY-MM'
  AND (installment_number IS NULL OR installment_number != 0)
```

### Sugestão Mensal de Meta

| Frequência | Cálculo |
|------------|---------|
| Mensal | `target_cents` (valor fixo por mês) |
| Anual | `target_cents / 12` |
| Por data | `target_cents / meses_restantes` |

### Auto-Alocação

Distribui `available_cents` entre gavetas com alocação zerada ou menor que a meta, na ordem escolhida pelo usuário. Para cada gaveta, aloca o mínimo entre o valor sugerido e o saldo disponível restante.

---

## Componentes Compartilhados

### `MainScaffold` (bottom_nav.dart)

Layout raiz do app. Inicializa todos os repositórios e BLoCs. Gerencia a tab selecionada e exibe o FAB central para nova transação.

### `AppCard` (app_card.dart)

Container com fundo e bordas arredondadas, cor adaptada ao tema.

```dart
AppCard(isDark: isDark, child: content)
AppCardTitle(label: 'Título', isDark: isDark)
```

### `AllocationBanner` (allocation_banner.dart)

Banner expansível exibido na Home com o saldo disponível para alocar. Ao expandir, mostra a lista de gavetas e botões de auto-alocação. Usa `OverlayEntry` para renderizar sobre os outros widgets.

### `BudgetPopoverButton` (budget_popover.dart)

Botão no canto superior direito da Home. Dropdown com lista de orçamentos, opção de criar novo orçamento e acesso às Configurações.

---

## Como Rodar

### Requisitos

- Flutter SDK 3.x
- Xcode 15+
- CocoaPods

### Setup

```bash
cd gaveta
flutter pub get
cd ios && pod install && cd ..
```

### Executar no simulador

```bash
flutter run
```

### Build para dispositivo físico

```bash
flutter build ios --debug
# ou
flutter build ios --release
```

> O app requer assinatura de código para rodar em dispositivo físico. Configure o Team ID no Xcode (`ios/Runner.xcworkspace`).

### Limpar build cache

```bash
flutter clean
flutter pub get
cd ios && pod install && cd ..
```
