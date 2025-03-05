#include <iostream>
#include <chrono>
#include <random>
#include <limits>
#include <algorithm>
#include <string>
#include <iomanip>
#include <fstream>
#include <clocale>
#include <vector>

using std::swap;


int losowa_liczba(int min, int max)
{
    static std::default_random_engine gen(std::random_device{}());
    static std::uniform_int_distribution<int> dist;
    return dist(gen, std::uniform_int_distribution<int>::param_type{ min, max });
}

void wypelnij_losowa_liczba(int* tablica, int rozmiar, int min = 0, int max = std::numeric_limits<int>::max())
{
    for (int i = 0; i < rozmiar; ++i)
        tablica[i] = losowa_liczba(min, max);
}

bool jest_posortowane(int* tablica, int rozmiar)
{
    return std::is_sorted(tablica, tablica + rozmiar);
}

double mierz_czas(int* tablica, int rozmiar, void(*funkcja_sortujaca)(int*, int))
{
    auto start = std::chrono::high_resolution_clock::now();
    funkcja_sortujaca(tablica, rozmiar);
    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double> duration = end - start;
    return duration.count();
}

/*

    parametry:
     - funkcja_sortujaca - wskanik na funkcjącą sortowanie, musi przyjmować dwa parametry: adres początku tablicy (int*) oraz jej rozmiar (int)
     - nazwa - nazwa testowanej funkcji, tylko w celach wypisania
     - output - strumien do ktorego beda zapisane wyniki, domyslnie std::cerr, przy ostatnim uruchomieniu warto nadpisac otwartym strumieniem plikowym, aby sobie zebrac wyniki do pliku
     - dodatkowe_miejsce - liczba dodatkowych elementow tablicy zaalokowanych PRZED poczatkiem tablicy, przykladowo gdy =1, pierwszym indeksem tablicy jest -1, ale dane rozpoczynaja sie od indeksu 0, moze sie przydac do sortowania przez wstawianie z wartownikiem
*/
void eksperyment(void(*funkcja_sortujaca)(int*, int), const std::string& nazwa, std::ostream& output = std::cerr, int dodatkowe_miejsce = 0)
{
    std::vector<double> srednie;
    //ustawienia
    const double limit_czasu = 25.0; //sekund
    const int powtorzen = 3;
    const int rozmiar_poczatkowy = 1000;
    /////////////////////////////////////////
    const int szerokosc = 100;
    int gwiazdek = szerokosc - nazwa.length() - 2;
    if (gwiazdek < 0)
        gwiazdek = 0;
    int i = 0;
    output << " ";
    for (; i < gwiazdek / 2; ++i)
        output << '*';
    output << " " << nazwa << " ";
    for (; i < gwiazdek; ++i)
        output << '*';
    output << "\n\n";
    output.flush();

    output << std::setw(9) << "N";
    output << std::setw(1) << "";
    for (int nr = 0; nr < powtorzen; ++nr)
        output << std::setw(9) << nr + 1 << " ";
    output << std::setw(12) << "średnia" << " ";
    output << "\n";
    for (int rozmiar = rozmiar_poczatkowy; ; rozmiar *= 2)
    {
        output << std::setw(9) << rozmiar << "\t";
        output.flush();
        int* pamiec = new int[dodatkowe_miejsce + rozmiar];
        int* tablica = pamiec + dodatkowe_miejsce;
        double czas = 0.0;

        int* pattern = new int[rozmiar];


        for (int nr = 0; nr < powtorzen; ++nr)
        {
            wypelnij_losowa_liczba(tablica, rozmiar);
            for (int i = 0; i < rozmiar; ++i)
                pattern[i] = tablica[i];
            std::sort(pattern, pattern + rozmiar);
            double c = mierz_czas(tablica, rozmiar, funkcja_sortujaca);
            if (!jest_posortowane(tablica, rozmiar))
            {
                output << "Tablica nieposortowana!!\n";
                if (&output != &std::cerr)
                    std::cerr << "Tablica nieposortowana!!\n";
                return;
            }
            if (!std::equal(pattern, pattern + rozmiar, tablica, tablica + rozmiar))
            {
                output << "Tablica zawiera inne wartosci niz powinna!!\n";
                if (&output != &std::cerr)
                    std::cerr << "Tablica zawiera inne wartosci niz powinna!!\n";
                return;
            }
            czas += c;
            output.precision(6);
            output << std::fixed << c << "  ";
            output.flush();
        }
        czas /= powtorzen;
        srednie.push_back(czas);
        output << std::fixed << czas << "\n";
        output.flush();
        delete[] pamiec;
        delete[] pattern;
        //koniec eksperymentu, jeśli średni czas przekroczył limit lub tablica >= 67108864
        if (czas > limit_czasu || rozmiar >= 67108864)
        {
            std::cout << "\nŚrednie czasy dla kolejnych rozmiarów tablicy:\n";
            for (double cz : srednie)
                std::cout << cz << ", ";
            break;
        }
    }
    output << "\n";
    output.flush();
}




void sortowanie_babelkowe(int* tablica, int rozmiar)
{
    
    for (int i = 0; i < rozmiar; i++) {
            for (int g = 1; g < rozmiar; g++) {
                if (tablica[g - 1] > tablica[g]) {
                    int tmp = tablica[g - 1];
                    tablica[g - 1] = tablica[g];
                    tablica[g] = tmp;
                }
            }
        }
        
}

    void sortowanie_wybieranie(int* tablica, int rozmiar)
    {
        for (int i = 0; i <= rozmiar - 2; i++) {
            int g = i;
            int j = tablica[i];
            for (int c = i + 1; c < rozmiar; c++) {
                if (tablica[c] < j) {
                    g = c;
                    j = tablica[c];
               }
            }
            tablica[g] = tablica[i];
            tablica[i] = j;
        }
    }

void sortowanie_wstawianie(int* tablica, int rozmiar)
{
    //TODO: zaimplementować
}

void sortowanie_babelkowe_mod(int* tablica, int rozmiar)
{
    //TODO: zaimplementować
}


int main()
{
    setlocale(LC_ALL, "");
    std::ofstream wyniki("wyniki.txt");

    std::ostream& output = std::cerr; //zmienic na = wyniki aby zapisywalo do pliku
    eksperyment(sortowanie_babelkowe, "Sortowanie bąbelkowe", output);
    eksperyment(sortowanie_wybieranie, "Sortowanie przez proste wybieranie", output);
    eksperyment(sortowanie_wstawianie, "Sortowanie przez proste wstawianie", output);
    eksperyment(sortowanie_babelkowe_mod, "Sortowanie bąbelkowe z modyfikacją", output);
    //...

    return 0;
}
