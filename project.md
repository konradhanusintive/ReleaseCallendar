Wymagania Techniczne: Aplikacja "Release Calendar"
1. Stos Technologiczny (Tech Stack)
   Backend: PHP (zalecana wersja 8.x). Struktura proceduralna lub proste MVC (w zależności od preferencji, bez wymogu ciężkich frameworków, jeśli ma być bardzo prosto).
   Baza Danych: MySQL (wersja 8.0+) / MariaDB. Komunikacja za pomocą biblioteki PDO (z bindowaniem parametrów dla bezpieczeństwa SQL Injection).
   Frontend: HTML5, CSS3, czysty JavaScript. (Opcjonalnie Bootstrap 5 lub Tailwind CSS dla szybkiego i estetycznego stworzenia widoku kalendarza i formularzy).
   Autoryzacja: Wbudowane sesje PHP ($_SESSION), hasła szyfrowane za pomocą funkcji password_hash() (np. BCRYPT).
2. Model Bazy Danych (Główne Tabele MySQL)
   Aplikacja będzie opierać się na kilku relacyjnych tabelach:
   users (Użytkownicy systemu)
   id (PK), username (login), password_hash, role (np. Admin, Release Manager, Developer, QA), created_at.
   releases (Główna tabela wydań)
   id (PK), version_name (np. v2.4.0), target_release_date, status (Draft, In Progress, Released, Cancelled), release_engineer_id (FK -> users).
   release_phases (Fazy konkretnego wydania - wyliczane na podstawie Blueprintu)
   id (PK), release_id (FK), phase_name (np. Feature Freeze), start_date, end_date, status (Pending, Done, Blocked).
   rollout_steps (Etapy publikacji dla wydania)
   id (PK), release_id (FK), percentage (1, 5, 10, 20, 50, 100), scheduled_date, status (Pending, Active, Completed).
   absences (Kalendarz urlopów/nieobecności)
   id (PK), user_id (FK), start_date, end_date, type (Urlop, L4 itp.).
3. Wymagania Funkcjonalne
   3.1. Moduł Autoryzacji i Użytkowników
   Rejestracja: Prosty formularz (Login, Hasło, Powtórz hasło). Hasło hashowane i zapisywane w bazie.
   Logowanie: Formularz (Login, Hasło). Weryfikacja przez password_verify(). Po zalogowaniu utworzenie sesji.
   Zarządzanie sesją: Brak dostępu do kalendarza bez zalogowania. Przycisk "Wyloguj" niszczący sesję.
   3.2. Główny Kalendarz (Widok Dashboardu)
   Widok miesięczny/tygodniowy (np. z użyciem lekkiej biblioteki JS typu FullCalendar lub własnej tabeli).
   Na kalendarzu zaznaczone są paski trwania konkretnych wydań (Releases).
   Na kalendarzu widoczne są również nieobecności zespołu (aby Release Manager wiedział, czy w czasie Sanity Tests nie brakuje kluczowych testerów lub programistów).
   3.3. Generator Wydania (Release Blueprint)
   System musi posiadać funkcję "Dodaj nowe wydanie", która działa jak szablon (Blueprint).
   Użytkownik podaje docelową datę wydania na produkcję (Target Release Date), a system automatycznie oblicza "wstecz" daty poszczególnych faz, rozpisując je w tabeli release_phases.
   Ścieżka faz (zgodnie z Twoim opisem), które system musi wygenerować i śledzić:
   Plan of Record (POR): Ustalenia ze stakeholderami, przypisanie zadań (Dev) i test case'ów (QA).
   Development Sprints: Faza programowania (Sprinty).
   Feature Freeze: Blokada nowych funkcji (tylko bugfixy).
   Regression Tests: Testy regresyjne przed Sanity.
   Sanity Tests (Staging & Prod):
   Testy na środowisku Staging.
   Testy na środowisku Produkcyjnym.
   Ograniczone czasowo (np. 1-2 dni), masowe testowanie przez Dev, QA, Menedżerów.
   Automated & Security Tests: Testy automatyczne, performance (wydajnościowe) i penetracyjne na środowisku Staging.
   Go / No-Go Meeting: Checklista/status decyzji zespołu.
   CAB (Change Advisory Board): Akceptacja dyrekcji/zarządu.
   Store Submission: Zaznaczenie wysłania aplikacji do App Store Connect i Google Play Console.
   Marketing Sync: Potwierdzenie aktualizacji materiałów przez dział marketingu.
   3.4. Moduł Rolloutu (Phased Rollout)
   Dla każdego wydania, które przejdzie fazę "Store Submission", system otwiera zakładkę "Rollout".
   Harmonogram rozłożony zazwyczaj na 7 dni.
   Checkboxy / Statusy dla kolejnych progów procentowych: 1% -> 5% -> 10% -> 20% -> 50% -> 100%.
   Możliwość wstrzymania rolloutu (Halt), jeśli pojawią się błędy.
   3.5. Moduł Post-Release (Support & Bug Triage)
   Sekcja w widoku wydania (aktywna w trakcie i po rolloucie) pozwalająca na zgłaszanie/śledzenie napływających ticketów z Service Desku.
   "Bug triage meeting" – miejsce na notatki, czy rollout jest kontynuowany, czy robimy hotfix.
   3.6. Role i Przypisania
   Każde wydanie musi mieć zdefiniowanego w bazie (przypisanego z listy) Release Engineera oraz Release Coordinatora.
   Dzięki temu każdy wie, kto odpowiada za "przeklikiwanie" statusów z Pending na Done.
4. Przykładowa struktura plików aplikacji (Propozycja)
   code
   Text
   /release-calendar
   │
   ├── /config
   │   └── database.php       # Połączenie z bazą PDO
   │
   ├── /includes
   │   ├── header.php         # Logo, nawigacja (jeśli zalogowany)
   │   ├── footer.php         # Zamknięcie tagów HTML
   │   └── auth.php           # Funkcje sprawdzające sesję
   │
   ├── index.php              # Główny widok kalendarza po zalogowaniu
   ├── login.php              # Formularz logowania i obsługa POST
   ├── register.php           # Formularz rejestracji
   ├── logout.php             # Zniszczenie sesji
   │
   ├── release_create.php     # Formularz dodawania nowego wydania (Blueprint)
   ├── release_view.php       # Widok szczegółowy wydania (odznaczanie faz, Go/No-go)
   ├── rollout_manage.php     # Odznaczanie procentów (1%...100%)
   └── absences.php           # Moduł zgłaszania/podglądu urlopów
5. Podsumowanie dla Programisty (Czego potrzebujesz do startu)
   Serwer: Zwykły hosting współdzielony (np. Linux, Apache) z obsługą PHP i MySQL zupełnie wystarczy.
   Logika dat (Kluczowy element): Programista musi użyć klasy DateTime w PHP, aby stworzyć algorytm Blueprintu. (np. $releaseDate->modify('-14 days') żeby wyznaczyć start Feature Freeze).
   Prostota interfejsu: Zamiast budować wszystko od zera w JS, wystarczy, że kliknięcie "Zakończ fazę Sanity" wyśle prosty formularz POST (przeładuje stronę) i zaktualizuje status w bazie MySQL (UPDATE release_phases SET status = 'Done' WHERE id = ?).
   Tak napisana specyfikacja pozwala od razu usiąść do pisania kodu w czystym PHP/MySQL. Czy chcesz, abym wygenerował dla Ciebie kod startowy dla połączenia z bazą danych oraz system rejestracji/logowania, abyś miał bazę do działania?/release-calendar
│
├── /config
│   └── database.php       # Połączenie z bazą PDO
│
├── /includes
│   ├── header.php         # Logo, nawigacja (jeśli zalogowany)
│   ├── footer.php         # Zamknięcie tagów HTML
│   └── auth.php           # Funkcje sprawdzające sesję
│
├── index.php              # Główny widok kalendarza po zalogowaniu
├── login.php              # Formularz logowania i obsługa POST
├── register.php           # Formularz rejestracji
├── logout.php             # Zniszczenie sesji
│
├── release_create.php     # Formularz dodawania nowego wydania (Blueprint)
├── release_view.php       # Widok szczegółowy wydania (odznaczanie faz, Go/No-go)
├── rollout_manage.php     # Odznaczanie procentów (1%...100%)
└── absences.php           # Moduł zgłaszania/podglądu urlopów