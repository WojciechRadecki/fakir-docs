# Konfiguracja schematu Businesslink

**Dotyczy:** importu dokumentów z usługi Businesslink do Fakira
Idea jest taka:
1. Badamy ustawioną klasyfikację i na jej podstawie ustalamy numer konta
2. Klasyfikacja może być ustawiona tylko w nagłówku, wtedy wszsytkie pozycje mają to samo konto
3. Klasyfikacja może być ustawiona dla pozycji. Wtedy na tych pozycjach, na których
jest ustawiona dekret zrobi się na jej podstawie. Na pozycjach bez wybranej
klasyfikacji ustawi się dekret na podstawie klasyfikacji z nagłówka.
Do ustalenia czy klasyfikacja jest na którejkolwiek pozycji służy zmienna [@jest_klasyfikacja_poz]

Kwota netto występuje w dwóch warunkach w zalezności, czy księgujemy tylko z nagłówka czy z pozycji


## Klasyfikacja - ustawiana w menu Dokumenty -> Import z KSeF i Businesslink

Przykład klasyfikacji oparty na tym, że kod jest bezpośrednio istniejącym kontem.
Znacznie ułatwia to późniejsze tworzenie schematu. Opis wspomaga obsługę.

![Klasyfikacja – przykład konfiguracji](images/klasyfikacja-przykład-konfiguracji.png)

## Schemat - menu Import definicje -> Import - schematy księgujące -> Nazwa źródła 'Businesslink'

### Najpierw wykonaj aktualizację metabazy żeby przesłać wpisane w Klasyfikacji dane do schematu

![Aktualizuj metabazę](images/metabaza-aktualizacja.png)

Tę operację należy powtarzać każdorazowo kiedy dodajemy coś do klasyfikacji

Usunięcie z metabazy skasowanych z klasyfikacji danych można zrobić po wejściu w metadefinicje

![Kasowanie wpisów z metabazy](images/metabaza-kasowanie-wpisow.png)

## teraz definiujemy zmienne

![alt text](zmienne.png)

### [@jest_klasyfikacja_poz]

![alt text](zmienne-jest-klasyfikacja-poz.png)

(to do dodania)

```sql
(select case when exists (select 1 from dbo.vMagikImpPozDokBL v where v.IMP_ID_DOKUMENTU = @imp_id_dokumentu and v.ID_FK_KOD_KSIEGOWY_TREE is not null) then 'Tak' else 'Nie' end)
```

### [@klasyfikacja_nag]

![alt text](zmienne-klasyfikacja-nag.png)

```sql
(select top 1 KOD_TREE from dbo.fn_fk_BL_ListaKodowKsiegowych ()
where ID_FK_KOD_KSIEGOWY_TREE = @d_id_kod_ksiegowy)
```

### [@klasyfikacja_poz]

![alt text](zmienne-klasyfikacja-poz.png)

(ten kod ewentualnie do poprawy)

```sql
(select top 1 KOD_TREE from dbo.fn_fk_BL_ListaKodowKsiegowych ()
where ID_FK_KOD_KSIEGOWY_TREE = @poz_id_kod_ksiegowy)
```

## Definicja warunków

Ddajemy zmienną [@jest_klasyfikacja_poz] na FZ i KFZ

![alt text](definicja-warunkow-jest-klasyfikacja-poz.png)

Można ją oprzeć o warunki z metabazy (wpisane ręcznie)

![alt text](definicja-warunkow-jest-klasyfikacja-poz-metabaza.png)

## Przechodzimy do schematu

![alt text](schemat-ogolnie.png)

Kwoty i brutto i VAT tradycyjnie, kwota netto przy księgowaniu z nagłówka

![alt text](schemat-ksiegowanie-naglowek.png)

Używamy tu w warunku zmiennej [jestklasyfikacja poz] zdefiniowanej krok wcześniej.
I kodu dla konta DT

```sql
case
    when isnull(@klasyfikacja_nag, '') = '' then '999'
    else @klasyfikacja_nag
end
```

I księgowanie z pozycji (tu ewentualnie algorytm do dodania\poprawy)

![alt text](schemat-ksiegowanie-pozycje.png)


```sql
case
    when nullif(@klasyfikacja_poz, '') is not null then @klasyfikacja_poz
    when nullif(@klasyfikacja_nag, '') is not null then @klasyfikacja_nag
    else '999'
end
```

# Przykłady

### W nagłowku jedna klasyfikacja, w dwóch pozycjach dwie kolejne, jednej brak

![alt text](example-010.png)

Dwa konta wzięte z pozycji, trzecie z nagłówka

![alt text](example-011.png)