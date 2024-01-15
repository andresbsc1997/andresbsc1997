- 👋 Hi, I’m @andresbsc1997
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...



pip install python-Andres
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from typing import Optional
from paypalhttp import PayPalHttpClient, OrdersCreateRequest

app = FastAPI()

# Configura las credenciales de PayPal
client_id = "tu_client_id_de_paypal"
client_secret = "tu_client_secret_de_paypal"

environment = PayPalHttpClient.SandboxEnvironment(client_id=client_id, client_secret=client_secret)
client = PayPalHttpClient(environment)

# Configuración de autenticación con tokens JWT
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

SECRET_KEY = "tu_clave_secreta"  # Reemplaza con una clave segura en un entorno de producción
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Función para generar tokens JWT
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# Endpoint para inicio de sesión y generación de token JWT
@app.post("/login")
async def login(username: str, password: str):
    # Aquí deberías verificar las credenciales del usuario en tu sistema
    # Simulamos un usuario válido para demostración
    if username == "usuario_demo" and password == "contraseña_demo":
        token_data = {"sub": username}
        access_token = create_access_token(token_data)
        return {"access_token": access_token, "token_type": "bearer"}
    else:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas",
            headers={"WWW-Authenticate": "Bearer"},
        )

# Endpoint protegido con autenticación para generar enlace de pago
@app.get("/generar_enlace_pago/{monto}")
async def generar_enlace_pago(
    monto: float,
    current_user: str = Depends(oauth2_scheme),
):
    try:
        # Aquí puedes utilizar el usuario autenticado (current_user) en tu lógica de generación de enlace de pago
        request = OrdersCreateRequest()
        request.prefer("return=representation")
        request.request_body(
            {
                "intent": "CAPTURE",
                "purchase_units": [{"amount": {"currency_code": "USD", "value": str(monto)}}],
            }
        )
        response = client.execute(request)
        approval_url = [link.href for link in response.result.links if link.rel == "approve"][0]
        return {"enlace_pago": approval_url}

    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error al generar enlace de pago: {str(e)}")

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="127.0.0.1", port=8000)
    

<!---
andresbsc1997/andresbsc1997 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
