# 🔐 AWS Secrets Manager Local Demo (amb Sidecar Simulat)

Aquest projecte mostra com simular l'ús d'**AWS Secrets Manager** en entorns **locals** utilitzant l'emulador [`skarpdev/aws-secrets-manager-emulator`](https://hub.docker.com/r/skarpdev/aws-secrets-manager-emulator).

## 🔑 Què és AWS Secrets Manager?
És un servei d’AWS que permet emmagatzemar, gestionar i rotar credencials sensibles de manera segura.

Què pots guardar-hi?

- Contrasenyes de bases de dades
- Claus d’API
- Tokens d’aplicacions
- Credencials de serveis externs

## Arquitectura

- **secrets-emulator**: contenidor que simula AWS Secrets Manager.
- **app**: sidecar.py (o script equivalent), que no està en el projecte però que pots crear tu. Aquest script és el que consulta secrets a l’emulador via API (usant AWS CLI o boto3).

Aquest comportament és el que simula el sidecar. Què seria un Sidecar (real)? En arquitectura de contenidors (Docker, Kubernetes...), un sidecar és un contenidor que: s’executa al costat del contenidor principal (la teua aplicació) i fa funcions de suport (logging, secrets, monitorització...)

És una forma de separar responsabilitats. La teua app no ha de gestionar directament secrets, només llegir arxius o fer peticions locals.

## Com executar

```bash
docker compose up --build
```

Ara pots accedir a la GUI en: `http://localhost:3000`

## Tutorial d’ús de la GUI (`http://localhost:3000`)

### 1. Obre la GUI

Accedeix a: `http://localhost:3000`

---

### 2. Crea un nou secret

1. Clica **Create**
2. Introdueix:
   - **SecretId**: `un-secret`
   - **SecretString** (format JSON):
     ```json
     {
       "db_host": "localhost",
       "db_user": "admin",
       "db_pass": "1234"
     }
     ```
3. Fes clic a **Create**

---

### 3. Verifica

Revisa que el secret aparega a la llista. El pots visualitzar, editar o eliminar.

---

### 4. Llegeix el secret amb `aws cli`

```bash
export AWS_ACCESS_KEY_ID=una-arreu
export AWS_SECRET_ACCESS_KEY=una-arreu
export AWS_DEFAULT_REGION=eu-west-1

aws secretsmanager get-secret-value \
  --secret-id un-secret \
  --endpoint-url http://localhost:3000 \
  --region eu-west-1
```

Resultat esperat:
```json
{
  "Name": "un-secret",
  "SecretString": "{\"db_host\":\"localhost\",\"db_user\":\"admin\",\"db_pass\":\"1234\"}"
}
```

---

### 5. Exporta el secret a fitxer

```bash
aws secretsmanager get-secret-value \
  --secret-id un-secret \
  --endpoint-url http://localhost:3000 \
  --region eu-west-1 \
  --query SecretString \
  --output text > un-secret.json
```

---

### 6. La teua app pot llegir el secret així:

```python
import json
with open("/secrets/un-secret.json") as f:
    secret = json.load(f)
print(secret["db_pass"])
```

---

## AWS Secrets Manager Emulator amb Precarrega de Secrets

Aquest projecte mostra com carregar secrets d'exemple a l'emulador `skarpdev/aws-secrets-manager-emulator` usant la carpeta `secrets-preload/`.

Secrets d'exemple:

- **db-config**: secret amb usuari i contrasenya (SecretString).
- **encrypted-key**: secret binari codificat en Base64 (SecretBinary).

## Limitacions conegudes

- No suporta totes les operacions (`ListSecrets`, `TagResource`...)
- No simula IAM ni control d'accés
- No hi ha autenticació
- No hi ha control de versions com a AWS real

---

## Notes

Aquest exemple és ideal per a testing local, CI/CD, o per entendre com funciona la integració de secrets

## Recursos

- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Docker Hub - skarpdev/aws-secrets-manager-emulator](https://hub.docker.com/r/skarpdev/aws-secrets-manager-emulator)


