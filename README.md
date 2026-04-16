# Quiosque Linux EBEG — Ansible (Fase 3 Zero Touch)

Provisionamento de quiosques Linux em modo non-interactive, orientado por inventario.

## Conceito da Fase 3

- Sem `vars_prompt` no playbook.
- Hostname e ID saem do alias do host no `inventory.ini`.
- URL e contexto de uso saem do `group_vars` do grupo correto.
- Execucao idempotente: pode rodar varias vezes sem "quebrar" o que ja esta aplicado.

## Regra de nome do host no inventario

Cada host deve seguir este padrao:

`kiosk-<unidade>-<uso>-<id>`

Exemplos validos:

- `kiosk-matriz-fac-01`
- `kiosk-matriz-cor-02`
- `kiosk-filial01-fac-01`

O playbook converte o alias para maiusculo e aplica como hostname no Linux.
Exemplo: `kiosk-matriz-fac-01` -> `KIOSK-MATRIZ-FAC-01`.

## Estrutura de grupos no inventory

O `inventory.ini` usa hierarquia:

- Grupos de uso: `matriz_fac`, `matriz_cor`, `filial_01_fac`, `filial_01_cor`
- Grupos de unidade com `:children`: `matriz`, `filial_01`
- Supergrupo `kiosks:children` com todas as unidades

## Onde configurar variaveis

- Globais: `group_vars/all.yml`
  - `kiosk_alias_prefix`
  - pacote/binario do Chromium
- Por grupo de uso:
  - `group_vars/matriz_fac.yml`
  - `group_vars/matriz_cor.yml`
  - `group_vars/filial_01_fac.yml`
  - `group_vars/filial_01_cor.yml`

Cada grupo de uso define:

- `kiosk_unidade_slug`
- `kiosk_tipo_slug`
- `kiosk_start_url`

## Passo a passo para Joao Victor (Analista)

### 1) Declarar uma nova maquina

Abrir `inventory.ini` e adicionar no grupo correto.

Exemplo: nova maquina da fabrica da Matriz:

```ini
[matriz_fac]
kiosk-matriz-fac-03 ansible_host=192.168.1.53 ansible_user=operador.ti
```

Exemplo: nova maquina corporativa da Filial 01:

```ini
[filial_01_cor]
kiosk-filial01-cor-02 ansible_host=192.168.20.42 ansible_user=operador.ti
```

### 2) Revisar URL do grupo

Verificar no arquivo do grupo se a URL esta correta:

- `group_vars/matriz_fac.yml` (SharePoint/producao)
- `group_vars/matriz_cor.yml` (portal corporativo)
- `group_vars/filial_01_fac.yml`
- `group_vars/filial_01_cor.yml`

### 3) Rodar deploy de uma maquina

```bash
ansible-playbook playbook.yml --limit kiosk-matriz-fac-03
```

### 4) Rodar deploy em todas as maquinas

```bash
ansible-playbook playbook.yml
```

## Idempotencia (importante)

Este projeto foi feito para ser idempotente:

- Rodar de novo nao duplica usuario, nao quebra LightDM e nao remove configuracao valida.
- So aplica mudanca quando houver diferenca.
- Pode executar em lote (todas as maquinas) com seguranca operacional.

## Observacoes de seguranca

- Nao coloque senha/chave em `inventory.ini` ou `group_vars`.
- Use Ansible Vault para segredos.
- `host_key_checking` esta desabilitado por decisao operacional interna (ambiente dinamico).
- Sandbox do Chromium continua ativo por padrao.
