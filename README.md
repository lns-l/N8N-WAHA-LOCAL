# N8N + WAHA + PostgreSQL + Redis - Ambiente Completo

Este projeto configura um ambiente completo para automação com WhatsApp usando N8N, WAHA (WhatsApp HTTP API), PostgreSQL e Redis.

## 🏗️ Arquitetura

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    WAHA     │    │     N8N     │    │  PostgreSQL │
│  WhatsApp   │◄──►│  Workflows  │◄──►│   Database  │
│   API       │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │    Redis    │
                   │    Cache    │
                   └─────────────┘
```

## 📋 Pré-requisitos

- Docker Desktop instalado
- Docker Compose instalado
- Portas disponíveis: 3000, 5050, 5432, 5678, 6379

## 🚀 Instalação e Configuração

### 1. Clone o Repositório
```bash
git clone <seu-repositorio>
cd N8N-WAHA-LOCAL
```

### 2. Inicie os Serviços
```bash
docker-compose up -d
```

### 3. Aguarde a Inicialização
Aguarde alguns minutos para todos os serviços inicializarem completamente.

## 🌐 Interfaces Web

### N8N (Automação Principal)
- **URL**: http://localhost:5678
- **Função**: Criar e gerenciar workflows de automação
- **Primeiro acesso**: Crie um usuário admin

### WAHA (WhatsApp)
- **URL**: http://localhost:3000
- **Função**: Gerenciar sessões do WhatsApp
- **Funcionalidades**: 
  - Conectar dispositivos WhatsApp
  - Visualizar sessões ativas
  - Gerenciar mídia
  - Ver logs de mensagens

### pgAdmin (Banco de Dados)
- **URL**: http://localhost:5050
- **Login**: admin@admin.com
- **Senha**: admin
- **Função**: Interface gráfica para gerenciar PostgreSQL

## 🔧 Configuração Inicial

### Configurando o N8N

1. **Acesse**: http://localhost:5678
2. **Crie um usuário admin** na primeira vez
3. **Configure o banco de dados** (já configurado automaticamente):
   - Tipo: PostgreSQL
   - Host: postgres
   - Port: 5432
   - Database: default
   - Username: default
   - Password: default

### Configurando o WAHA

1. **Acesse**: http://localhost:3000
2. **Crie uma nova sessão** do WhatsApp
3. **Escaneie o QR Code** com seu WhatsApp
4. **Aguarde a conexão** ser estabelecida

### Configurando o pgAdmin

1. **Acesse**: http://localhost:5050
2. **Faça login** com admin@admin.com / admin
3. **Adicione o servidor PostgreSQL**:
   - Nome: N8N Database
   - Host: postgres
   - Port: 5432
   - Database: default
   - Username: default
   - Password: default

## 📱 Criando Workflows no N8N

### Exemplo: Workflow de Mensagem WhatsApp

1. **Crie um novo workflow** no N8N
2. **Adicione um trigger Webhook**:
   - URL: `/webhook/webhook`
   - Método: POST
3. **Adicione um nó HTTP Request** para enviar mensagem:
   - URL: `http://waha:3000/api/sendText`
   - Método: POST
   - Headers: `Content-Type: application/json`
   - Body:
   ```json
   {
     "session": "sua-sessao",
     "to": "{{$json.from}}",
     "text": "Mensagem automática recebida!"
   }
   ```

## 🔄 Comandos Úteis

### Gerenciar Serviços
```bash
# Iniciar todos os serviços
docker-compose up -d

# Parar todos os serviços
docker-compose down

# Ver logs de um serviço específico
docker-compose logs n8n
docker-compose logs waha
docker-compose logs postgres

# Reiniciar um serviço específico
docker-compose restart n8n
```

### Backup e Restore
```bash
# Backup do banco de dados
docker exec n8n-waha-local-postgres-1 pg_dump -U default default > backup.sql

# Restore do banco de dados
docker exec -i n8n-waha-local-postgres-1 psql -U default default < backup.sql

# Backup dos volumes
docker run --rm -v n8n-waha-local_pgdata:/data -v $(pwd):/backup alpine tar czf /backup/pgdata-backup.tar.gz -C /data .
```

### Monitoramento
```bash
# Ver status dos containers
docker-compose ps

# Ver uso de recursos
docker stats

# Acessar Redis CLI
docker exec -it n8n-waha-local-redis-1 redis-cli
```

## 📊 Volumes Persistentes

Os seguintes dados são persistidos:

- **`redis_data`**: Cache e sessões do Redis
- **`pgdata`**: Banco de dados PostgreSQL
- **`waha_sessions`**: Sessões do WhatsApp
- **`waha_media`**: Mídia do WhatsApp
- **`n8n_data`**: Workflows e configurações do N8N
- **`pgadmin_data`**: Configurações do pgAdmin

## 🔒 Segurança

### Credenciais Padrão
- **PostgreSQL**: default/default
- **Redis**: default/default
- **pgAdmin**: admin@admin.com/admin

### Recomendações
1. **Altere as senhas** em produção
2. **Use HTTPS** em ambiente de produção
3. **Configure firewall** adequadamente
4. **Monitore logs** regularmente

## 🐛 Troubleshooting

### Problemas Comuns

#### N8N não acessível
```bash
# Verificar logs
docker-compose logs n8n

# Verificar se o banco está rodando
docker-compose ps postgres
```

#### WAHA não conecta
```bash
# Verificar logs
docker-compose logs waha

# Verificar se o N8N está rodando
docker-compose ps n8n
```

#### Banco de dados não conecta
```bash
# Verificar logs
docker-compose logs postgres

# Testar conexão
docker exec -it n8n-waha-local-postgres-1 psql -U default -d default
```

### Logs Úteis
```bash
# Ver todos os logs
docker-compose logs

# Ver logs em tempo real
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f n8n
```

## 📈 Monitoramento

### Métricas Importantes
- **Uso de CPU e RAM** dos containers
- **Espaço em disco** dos volumes
- **Logs de erro** dos serviços
- **Status das sessões** do WhatsApp

### Ferramentas de Monitoramento
- **Docker Stats**: `docker stats`
- **Logs**: `docker-compose logs`
- **pgAdmin**: Monitoramento do banco
- **N8N**: Dashboard interno

## 🔄 Atualizações

### Atualizar Imagens
```bash
# Parar serviços
docker-compose down

# Atualizar imagens
docker-compose pull

# Reiniciar com novas imagens
docker-compose up -d
```

### Backup Antes de Atualizar
```bash
# Backup completo
docker-compose down
tar czf backup-$(date +%Y%m%d).tar.gz .
docker-compose up -d
```

## 📞 Suporte

Para problemas ou dúvidas:
1. Verifique os logs dos serviços
2. Consulte a documentação oficial:
   - [N8N Documentation](https://docs.n8n.io/)
   - [WAHA Documentation](https://waha.devlike.pro/)
   - [PostgreSQL Documentation](https://www.postgresql.org/docs/)

## 📝 Licença

Este projeto é de uso livre para fins educacionais e comerciais.

---

**Desenvolvido com ❤️ para automação de WhatsApp** 

## 🗄️ Acessando o Redis

### 1. De dentro dos containers (N8N, WAHA, etc.)
- **Host:** `redis`
- **Porta:** `6379`
- **Usuário:** `default` (se necessário)
- **Senha:** `default`
- **String de conexão:**
  ```
  redis://:default@redis:6379
  ```

### 2. Do seu computador (host/Windows)
- **Host:** `localhost` ou `127.0.0.1`
- **Porta:** `6379`
- **Senha:** `default`
- **Comando CLI:**
  ```bash
  redis-cli -h 127.0.0.1 -p 6379 -a default
  ```

### 3. De uma aplicação externa
- **Host:** IP do seu computador na rede local
- **Porta:** `6379`
- **Senha:** `default`

> **Importante:**
> - O acesso interno entre containers deve sempre usar o nome do serviço (`redis`).
> - O acesso externo (host) usa `localhost` ou o IP da máquina.

--- 