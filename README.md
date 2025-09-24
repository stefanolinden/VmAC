# VmAC
# Referência rápida + Exemplos

## Tabela de comandos

| Comando                     | Uso (sintaxe)                                                     | Observações                                                                                       |
| --------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **ligar**                   | `ligar ar <valor>;`                                               | **Obrigatório** para iniciar ações num bloco. `<valor>` pode ser número ou variável.              |
| **modo**                    | `modo <cool\|heat\|auto\|fan>;`                                   | Opcional após `ligar`/`usar`.                                                                     |
| **vento**                   | `vento <baixo\|medio\|alto\|auto>;`                               | Opcional após `ligar`/`usar`.                                                                     |
| **por**                     | `por <valor> <min\|h>;`                                           | Opcional. **No máx. 1** por ativação (após `ligar` **ou** `usar`). Agenda desligar automático.    |
| **esperar**                 | `esperar <int> <min\|h>;`                                         | Pausa o fluxo do bloco atual. Use **após** `ligar`/`usar`.                                        |
| **desligar**                | `desligar;`                                                       | Encerra a ativação atual (opcional se houve `por`).                                               |
| **salvar**                  | `salvar <int> como <IDENT>;`                                      | Declara variável global numérica.                                                                 |
| **preset (definição)**      | `preset <IDENT> { <action_seq> }`                                 | Define combo reutilizável. O corpo **deve começar** com `ligar ar ...;`. Pode ter `por`.          |
| **usar (preset)**           | `usar <IDENT>;`                                                   | Aplica um preset. Pode ter cauda com `por`/`esperar`/`desligar` (respeitar 1 `por` por ativação). |
| **as (horário)**            | `as HH:MM { <action_or_preset> }`                                 | Executa no horário. Sem `por`/`desligar` ⇒ fica ligado.                                           |
| **das … ate … (período)**   | `das HHhMM ate HHhMM { <action_or_preset> }`                      | Liga no início; **desliga no fim** do período.                                                    |
| **se … entao … \[senao …]** | `se <cond> entao <ação\|{ bloco }>` `[ senao <ação\|{ bloco }> ]` | Ação pode ser: `usar`, `ligar`, `esperar`, `desligar` ou bloco `{ … }`.                           |
| **repetir**                 | `repetir <int> { <loop_body> }`                                   | Repete N vezes.                                                                                   |
| **enquanto**                | `enquanto <cond> { <loop_body> }`                                 | Repete enquanto condição for verdadeira.                                                          |
| **Sensores**                | `temp`, `temp_sala`, `hora`, `min`                                | Usáveis em condições/expressões.                                                                  |
| **Expressões**              | `<valor>` \| `<sensor>` \| `(<expr>)` \| `expr + - * / expr`      | Para `cond`: `== != < <= > >=` e conectivos `e`/`ou`.                                             |

> **Regra de ouro:** todo **`action_seq`** começa com `ligar ar …;` (ou use `usar <preset>;`, cujo corpo já tem `ligar`).

---

## Exemplos prontos

### 1) Imediato (sem horário)

```aclite
ligar ar 23;
```

```aclite
ligar ar 24;
modo cool;
vento medio;
```

```aclite
ligar ar 22;
por 45 min;
```

```aclite
ligar ar 24;
esperar 30 min;
desligar;
```

---

### 2) Horário (as HH\:MM)

```aclite
as 19:00 {
  ligar ar 23;
}
```

```aclite
as 18:30 {
  ligar ar 24;
  modo cool;
  vento baixo;
  por 2 h;
}
```

---

### 3) Período (das … ate …)

```aclite
das 08h00 ate 11h30 {
  ligar ar 24;
  modo cool;
  vento medio;
}
```

```aclite
das 14h00 ate 18h00 {
  ligar ar 23;
  por 1 h;
  esperar 30 min;
  ligar ar 23;
  por 1 h;
}
```

---

### 4) Variáveis

```aclite
salvar 23 como conforto;
ligar ar conforto;
```

```aclite
salvar 2 como horas;
ligar ar 24;
por horas h;
```

---

### 5) Presets

```aclite
preset noite {
  ligar ar 22;
  modo cool;
  vento baixo;
}

usar noite;
```

```aclite
preset soneca {
  ligar ar 24;
  por 2 h;
}

usar soneca;      // desliga sozinho em 2 h
```

```aclite
preset qualquer {
  ligar ar 25;
  modo cool;
  vento medio;
}

usar qualquer;
por 45 min;       // duração externa aplicada ao preset
```

```aclite
as 19:00 { usar noite; }
```

```aclite
das 09h00 ate 12h00 { usar noite; }
```

```aclite
usar noite;
esperar 30 min;
desligar;
```

---

### 6) Condicionais

```aclite
se temp > 26 entao usar noite;
```

```aclite
se temp_sala >= 28 entao ligar ar 23;
```

```aclite
se temp > 28 entao esperar 1 h;
```

```aclite
se temp > 28 entao usar soneca; senao esperar 1 h;
```

```aclite
se temp > 26 entao {
  ligar ar 23;
  vento medio;
  por 1 h;
} senao {
  usar noite;
  esperar 30 min;
}
```

```aclite
se temp >= 27 entao {
  usar qualquer;
  por 45 min;
  esperar 30 min;
}
```

```aclite
as 18:30 {
  se temp > 25 entao usar noite;
}
```

```aclite
das 14h00 ate 18h00 {
  se temp > 26 entao {
    ligar ar 23;
    por 1 h;
  } senao {
    esperar 30 min;
  }
}
```

---

### 7) Loops

```aclite
preset soneca {
  ligar ar 24;
  por 2 h;
}

repetir 3 {
  usar soneca;     // ativa 2 h e desliga sozinho
  esperar 30 min;  // intervalo entre repetições
}
```

```aclite
repetir 2 {
  ligar ar 23;
  por 1 h;
  esperar 30 min;
}
```

```aclite
enquanto temp > 28 {
  ligar ar 23;
  por 1 h;
  esperar 30 min;
}
```

```aclite
repetir 5 {
  se temp < 22 entao esperar 30 min;
  senao usar noite;
}
```

---

### 8) Combinações

```aclite
salvar 23 como conforto;

preset trabalho {
  ligar ar conforto;
  vento medio;
}

as 09:00 { usar trabalho; }
```

```aclite
preset leitura {
  ligar ar 25;
  vento baixo;
  modo cool;
}

usar leitura;
por 90 min;
esperar 30 min;
ligar ar 24;
por 1 h;
```

```aclite
das 10h00 ate 14h00 {
  usar soneca;     // por 2 h
  esperar 30 min;
  usar soneca;     // por 2 h (o período encerra às 14h00)
}
```



