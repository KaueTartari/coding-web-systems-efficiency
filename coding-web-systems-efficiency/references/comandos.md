# Comandos enxutos por ferramenta

Exemplos para adaptar. Troque nomes de tabela, arquivo, serviço e horário. Leia só a seção da ferramenta em uso.

## Sumário
1. Banco de dados (Postgres, MySQL, SQLite, Supabase, MongoDB, Firebase)
2. Logs (arquivos, Docker, systemd, PM2, Nginx, Vercel)
3. Dados (CSV, JSON, planilhas)
4. Testes (JS/TS, Python, PHP)
5. Build, install e git

---

## 1. Banco de dados

### Postgres / Supabase
```bash
psql "$DATABASE_URL" -c '\dt'                          # lista tabelas
psql "$DATABASE_URL" -c '\d pedidos'                   # colunas e tipos de uma tabela
psql "$DATABASE_URL" -c "SELECT id, status, created_at FROM pedidos WHERE status='erro' ORDER BY created_at DESC LIMIT 10"
psql "$DATABASE_URL" -c "SELECT status, COUNT(*) FROM pedidos GROUP BY status"
psql "$DATABASE_URL" -c "EXPLAIN ANALYZE SELECT ..."
```

### MySQL / MariaDB
```bash
mysql -e 'SHOW TABLES' banco
mysql -e 'DESCRIBE pedidos' banco
mysql -e "SELECT id, status FROM pedidos WHERE status='erro' LIMIT 10" banco
```

### SQLite
```bash
sqlite3 app.db '.tables'
sqlite3 app.db '.schema pedidos'
sqlite3 app.db "SELECT id, status FROM pedidos LIMIT 10"
```

### MongoDB
```bash
mongosh "$MONGO_URL" --quiet --eval 'db.getCollectionNames()'
mongosh "$MONGO_URL" --quiet --eval 'db.pedidos.find({status:"erro"},{_id:1,status:1}).limit(10).toArray()'
mongosh "$MONGO_URL" --quiet --eval 'db.pedidos.countDocuments({status:"erro"})'
```

### Firebase / Firestore
Prefira ler as regras e o código que grava os documentos. Para dados, use o SDK com `.limit(10)` e `.select(...)` num script, nunca exporte a coleção inteira.

### Dumps e backups
```bash
ls -lh backup.sql                                      # tamanho antes de qualquer coisa
rg -n 'CREATE TABLE' backup.sql | head -50             # só a estrutura
rg -n 'termo-procurado' backup.sql | head -20
```

---

## 2. Logs

### Arquivo de log
```bash
tail -n 100 storage/logs/app.log
rg -n 'ERROR|Exception|FATAL' app.log | tail -50
rg -n '2026-09-14 14:(2|3)' app.log | rg -i error      # janela de horário
rg -o 'ERROR.*' app.log | sort | uniq -c | sort -rn | head -20   # agrupa erros repetidos
```

### Docker
```bash
docker ps --format '{{.Names}}\t{{.Status}}'
docker logs --since 30m --tail 100 nome-container 2>&1 | rg -i 'error|exception'
docker compose logs --tail 100 api
```

### systemd / Linux
```bash
journalctl -u nome-servico --since '1 hour ago' -p err --no-pager | tail -50
```

### PM2 (Node)
```bash
pm2 logs nome-app --lines 100 --nostream --err
```

### Nginx / Apache
```bash
tail -n 50 /var/log/nginx/error.log
awk '$9 >= 500' /var/log/nginx/access.log | tail -20   # respostas 5xx
```

### Vercel / Netlify / serviços hospedados
Use o CLI com filtro de tempo e quantidade (ex.: `vercel logs <url> --since 1h`) e confirme as flags com `--help`, pois mudam entre versões.

---

## 3. Dados

### CSV
```bash
wc -l dados.csv
head -n 6 dados.csv
python3 -c "import pandas as pd; df=pd.read_csv('dados.csv'); print(df.shape); print(df.dtypes); print(df.head())"
python3 -c "import pandas as pd; df=pd.read_csv('dados.csv'); print(df.groupby('status')['valor'].agg(['count','sum']))"
```

### JSON
```bash
jq 'keys' resposta.json
jq '.data | length' resposta.json
jq '.data[0]' resposta.json
jq '[.data[] | select(.status=="erro") | {id, mensagem}] | .[:10]' resposta.json
curl -s https://api.exemplo.com/itens | jq '.[0]'
```

### Planilhas (.xlsx)
```bash
python3 -c "import pandas as pd; x=pd.ExcelFile('planilha.xlsx'); print(x.sheet_names)"
python3 -c "import pandas as pd; df=pd.read_excel('planilha.xlsx', sheet_name='Vendas'); print(df.shape); print(df.head())"
```

---

## 4. Testes

### JavaScript / TypeScript
```bash
npx vitest run src/utils/preco.test.ts --reporter=dot
npx vitest related src/utils/preco.ts --run            # só testes ligados ao arquivo alterado
npx jest src/utils/preco.test.ts --silent
npx jest -t 'calcula desconto' --silent
npx playwright test tests/checkout.spec.ts --reporter=line
```

### Python
```bash
pytest tests/test_pedidos.py -q
pytest -k 'desconto' -q --tb=short
pytest -q --lf                                         # só os que falharam da última vez
```

### PHP / Laravel
```bash
php artisan test --filter=PedidoTest
vendor/bin/phpunit --filter testCalculaDesconto
```

---

## 5. Build, install e git

```bash
npm run build 2>&1 | tail -30
npm run lint 2>&1 | rg -i 'error' | head -30
npx tsc --noEmit 2>&1 | head -30
npm install --silent 2>&1 | tail -10
git status --short
git diff --stat
git diff -- src/arquivo.ts
git log --oneline -10 -- src/arquivo.ts
```
