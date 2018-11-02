# 1. Historyjka Techsoup #
# 1.1 Aplikacja Javascript komunikująca się po HTTP z API.
# 1.2 Zabezpieczenie API przez zaszycie w kodzie js GUIDa
# 1.3 Sprawdzanie w API czy ten GUID jest dołączony do każdego request'a.
# 2. (Problem) Jak właściwie zabezpieczyć takie API prawidłowo?
# 3. Stwórzmy bazę danych, a w niej tabelę z użytkownikami i ich hasłami.
# 4. (Problem) Gdzie miałaby być ta baza? Razem z API? To wymagałoby żeby przesyłać login i hasło użytkownika z każdym requestem. To chyba do bani.
# 5. Zastosujmy Oauth2.
# 6. (Problem) Oauth ma flowy? Może Resource-Owner Password Credentials Flow?
# 7. Nie nadaje się.
# 8. (Problem) To może Client Crednetials Flow?
# 9. Lepiej ale nie ma danych o użytkowniku.
# 10. (Problem) To może Auth Code Flow?
# 11. Nie nadaje się bo to jest dla aplikacji server-side.
# 12. (Problem) To może Implicit Flow?
# 13. To jest to!
# 14. (Problem) Co z danymi o użytkowniku? 
# 15. OpenId.
# 16. (Problem) Jak pogodzić OpenId z Oauth2?
# 17. OpenId Connect.
