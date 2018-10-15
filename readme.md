# Notatki

## Rodzaje ataków

### COnfused deputy problem

Prawdziwa aplikacja (confused deputy) akceptuje token od wydany dla złośliwej aplikacji. **Atak możliwy tylko w przypadku implicit flow i w przypadku stosowania Oauth2 do uwierzytelniania**. Z powodu tego ataku ważna staje się walidacja tokenów poprzez sprawdzanie pola audience względem client_id. 
