---
layout:     post
title:      "User management service"
subtitle:   "manage user data, credential, token"
date:       2020-11-22 13:22:00
author:     "Xiaoxianma"
header-img: "img/home-bg-oakley-on-keyboard.jpg"
tags:
    - web
    - webserver
    - user
    - fastapi
    - token
    - python
---

Let's go through how to create tables, rest apis of users in fastapi.

## Create a user DB Model

Note: database only stores hashed password instead of real password.
```python
from app.db.session import Base
from sqlalchemy import Boolean, Column, Integer, String
from sqlalchemy_utils import generic_repr


@generic_repr
class User(Base):
    __tablename__ = "user"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    first_name = Column(String)
    last_name = Column(String)
    hashed_password = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
```

## Authentication

### 1. Signup

Signup a user account by providing **email** and **password**.  
Once server receives password, it stores hash value of the password to the user table by `passlib` library.  
```python
@r.post("/signup", summary="Signup new non-super user")
async def signup(db=Depends(get_db), form_data: OAuth2PasswordRequestForm = Depends()):
    user = sign_up_new_user(db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Account already exists",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return create_user_token(user)


def sign_up_new_user(db, email: str, password: str):
    user = get_user_by_email(db, email)
    if user:
        return False  # User already exists
    new_user = create_user(
        db,
        user_schema.UserCreate(
            email=email,
            password=password,
            is_active=True,
            is_superuser=False,
        ),
    )
    return new_user


def create_user(db: Session, user: user_schema.UserCreate):
    hashed_password = get_password_hash(user.password)
    db_user = user_model.User(
        first_name=user.first_name,
        last_name=user.last_name,
        email=user.email,
        is_active=user.is_active,
        is_superuser=user.is_superuser,
        hashed_password=hashed_password,
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### 2. Token creation

In the end of signup, an **access token** is created.  
Data `{'email': null, 'permission': null, 'exp': null}` is encoded by `jwt` library.  
JWT encoding requires server `SECRET_KEY` and an algorithm like `HS256`.
```python
def create_user_token(user: user_model.User):
    access_token_expires = timedelta(minutes=security.ACCESS_TOKEN_EXPIRE_MINUTES)
    if user.is_superuser:
        permissions = "admin"
    else:
        permissions = "user"
    access_token = security.create_access_token(
        data={"sub": user.email, "permissions": permissions},
        expires_delta=access_token_expires,
    )
    return {"access_token": access_token, "token_type": "bearer"}


def create_access_token(*, data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

### 3. Refresh token

By default, generated tokens are expired in 15 minutes. A refreshed token is generated everytime that user login.  
First, authenticate user by provided **email** and **password**. Using the same `passlib` to verify `plain_password` against `hashed_password` in the `user` table.  
Then, if authenticate successfully, a new user token is generated.
```python
@r.post("/token", summary="Refresh user token when login")
async def login(db=Depends(get_db), form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return create_user_token(user)


def authenticate_user(db, email: str, password: str):
    user = get_user_by_email(db, email)
    if not user:
        return False
    if not security.verify_password(password, user.hashed_password):
        return False
    return user


def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

## Auth REST API

Apply `AuthDependency` to each rest api.
```python
@self.r.get(
    f"/{self.ENDPOINT}/" + "{inst_id}",
    response_model=self.GET_SCHEMA_OUT,
)
async def get(
    inst_id: int,
    user: user_schema.UserBase = Security(AuthDependency()),
    db=Depends(get_db),
):
    return crud_utils.get_item(db, self.MODEL, inst_id)
```

STEPS to verify request is authenticated:  
1. Each request goes into `AuthDependency` dependency first. 
2. Get token from the header of the request.  
3. Try to decode token into data=`{'email': null, 'permission': null}`. Because the token has `exp`, if exp is expired, jwt will throw ExpiredSignatureError. Then server raises unauthorized exception.  
4. Validate email from data against `user` table. If user doesn't exist, server raises forbidden exception.  
5. Validate permission from data. If the permission is not allowed for this rest api, server raises forbidden exception.  
6. All looks good, validation is passing!
```python
class AuthDependency:
    def __init__(self, enabled: bool = True, token: t.Optional[str] = None):
        self.enabled = enabled
        self.token = token

    def raise_unauthorized_exception(self, authenticate_value):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials",
            headers={"WWW-Authenticate": authenticate_value},
        )

    def raise_forbidden_exception(self, authenticate_value):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough permissions",
            headers={"WWW-Authenticate": authenticate_value},
        )

    async def __call__(
        self,
        request: Request,
        security_scopes: SecurityScopes,
        db=Depends(session.get_db),
    ):
        from app.db.crud.user import get_user_by_email

        if not self.enabled:
            return None
        if security_scopes.scopes:
            authenticate_value = f'Bearer scope="{security_scopes.scope_str}"'
        else:
            authenticate_value = "Bearer"
        token: str = await oauth2_scheme(request) if not self.token else self.token
        if token is None:
            self.raise_unauthorized_exception(authenticate_value)
        try:
            payload = jwt.decode(token, security.SECRET_KEY, algorithm=[security.ALGORITHM])
        except (jwt.PyJWTError, jwt.ExpiredSignatureError):
            self.raise_unauthorized_exception(authenticate_value)
        email: str = payload.get("sub")
        permission: str = payload.get("permissions")
        user = get_user_by_email(db, email)
        if user is None:
            self.raise_forbidden_exception(authenticate_value)
        if permission is None or permission not in ["admin", "user"]:
            self.raise_forbidden_exception(authenticate_value)
        return user
```