# geb-kiosk — Ansible (arquitetura modular)

Provisionamento de quiosque Linux com **variáveis semânticas por grupo** no inventário `hosts.ini`. A URL do navegador é sempre **`url_kiosk`** (nome agnóstico de contexto: Fábrica, RH, Campanhas, etc.).

## Arquivos principais

| Arquivo | Função |
|---------|--------|
| `hosts.ini` | Grupos `[grupo:children]`, hosts e `[grupo:vars]` com `url_kiosk` |
| `kiosk_setup.yml` | Playbook: apt update, Chrome via `.deb`, LightDM + Cinnamon, `kiosk.desktop` |
| `ansible.cfg` | `inventory = hosts.ini` |
| `files/` | Coloque `google-chrome-stable_current_amd64.deb` (veja `files/README.md`) |

## Como o João Victor roda o deploy

Na raiz do repositório:

```bash
ansible-playbook kiosk_setup.yml
```

Ou limitando a um host:

```bash
ansible-playbook kiosk_setup.yml --limit VGP-QKFBDKPD001
```

## Nova máquina

1. Escolha o grupo de contexto (ex.: `quiosques_fabrica` ou `quiosques_rh`).
2. Adicione a linha do host com `ansible_host=...`.
3. Ajuste `url_kiosk` no bloco `[esse_grupo:vars]` correspondente.

## Idempotência

O playbook pode ser executado várias vezes: pacotes em estado `present`, arquivos de configuração sobrescritos de forma controlada e handler do LightDM só quando o snippet mudar.

## Segurança

Senhas no `hosts.ini` são convenientes para lab; em produção use **Ansible Vault** ou variáveis injetadas pelo pipeline, sem commitar segredos.
