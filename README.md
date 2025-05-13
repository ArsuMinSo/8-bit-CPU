# 8bitový procesor – popis architektury a instrukční sady

## **Architektura a komponenty**

Tento 8bitový procesor je navržen podle von Neumannovy architektury. Obsahuje jednoduchou sběrnici, instrukční registr, ALU, řadič a RAM. Vše je sestaveno ze základních logických hradel.

### **Sběrnice**
- **8bitová W Bus**: Jediná datová sběrnice spojující všechny hlavní komponenty.

### **Programový čítač (PC)**
- **4bitový registr**.
- **Signály**:
  - `Cp`: přičte 1 (inkrementace),
  - `Ep`: pošle hodnotu na W Bus,
  - `Le`: načte hodnotu do čítače (např. při skoku),
  - `CLK`, `CLR`: taktování a nulování.

### **Registry**
- **8bitové registry A a B**:
  - `Ea`: pošle obsah A na W Bus,
  - `La`: načte z W Bus do A,
  - `Lb`: načte z W Bus do B.

### **ALU (Aritmeticko-logická jednotka)**
- Operace: sčítání, odčítání (8bitové).
- Řízení: `Op` (0 = ADD, 1 = SUB),
- Výstup na sběrnici: `Eu`.

### **Výstupní registr**
- **8bitový Out**: výstupní registr řízený signálem `Lo`.

### **Adresový registr (MAR)**
- **4bitový**.
- `Lm`: načte adresu z W Bus.

### **RAM**
- **16×8bitová paměť**.
- `Lr`: uloží obsah W Bus na adresu danou MAR,
- `Er`: čte data z dané adresy na W Bus.

### **Instrukční registr (IR)**
- **8bitový**.
- `Li`: načte instrukci z W Bus,
- Horní 4 bity → řadič,
- Spodní 4 bity → W Bus.

### **Řadič (Control Unit)**
- Generuje signály pro všechny komponenty.
- Využívá **6bitový ring counter**, řízený signály:
  - `Cr`: restart počítadla (T1),
  - `Lc`: přímé načtení hodnoty do PC.

---

## **Instrukční sada**

| Binární     | Hex  | Mnemonik   | Formát | Popis                                  |
|-------------|------|------------|--------|----------------------------------------|
| `0000 xxxx` | 0x00 | `NOP`      | 1 B    | No operation                           |
| `0001 xxxx` | 0x10 | `LDA $h`   | 1 B    | Load from address $h into register A   |
| `0010 xxxx` | 0x20 | `STA $h`   | 1 B    | Store register A into address $h       |
| `0011 xxxx` | 0x30 | `ADD $h`   | 1 B    | Add value at $h to register A          |
| `0100 xxxx` | 0x40 | `SUB $h`   | 1 B    | Subtract value at $h from register A   |
| `1101 xxxx` | 0xD0 | `JMP $h`   | 1 B    | Jump to address $h                     |
| `1110 xxxx` | 0xE0 | `OUT`      | 1 B    | Output content of register A           |
| `1111 xxxx` | 0xF0 | `HLT`      | 1 B    | Halt processor                         |

Nepoužité kódy (např. `0x50`–`0xC0`) jsou rezervované a zatím slouží jako `NOP`.

---

## **Řízení instrukcí – Taktovací cykly**

Řadič má 6 vnitřních kroků (T1–T6), během nichž nastavuje aktivní signály.

| Instrukce    | T1                  | T2                         | T3                 | T4                 | T5                | T6         |
|--------------|---------------------|-----------------------------|---------------------|---------------------|-------------------|------------|
| `LDA $h`     | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Lm, Ei`            | `Er, La`            | `Cr`              | —          |
| `STA $h`     | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Lm, Ei`            | `Ea, Lr`            | `Cr`              | —          |
| `ADD $h`     | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Lm, Ei`            | `Er, Lb`            | `Op, Eu, La`      | `Cr`       |
| `SUB $h`     | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Lm, Ei`            | `Er, Lb`            | `Op, Eu, La`      | `Cr`       |
| `JMP $h`     | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Lc, Ei`            | `Cr`                | —                 | —          |
| `OUT`        | `Ep, Lm`            | `Cp, Er, Li, Cr`            | `Ea, Lo`            | `Cr`                | —                 | —          |
| `HLT`        | `Cr`                | —                           | —                   | —                   | —                 | —          |
