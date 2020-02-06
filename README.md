#include <iostream>
#include <fstream>
using namespace std;
//Entrada: edat(int), districte(int), compra(double), hora entrada/sortida(int hhmm)
//Sortida: resum compres (edat,preu,minuts permanencia)(mateix ordenat per edat)
//         despesa per minut i mostra si algu amb una edat X ha comprat
const int EOS = 0;
const int MAX = 100;

struct Compra{
	int edat;
	double preu;
	int minuts;
};

typedef Compra Taula_Compres[MAX];

void intercanvi(Compra &a, Compra &b)
{
	Compra aux = a;
	a = b;
	b = aux;
}

bool cerca_dicotomica_booleana(const Taula_Compres taula, int n, int numero)
{
	bool existeix = false;
	int esq = 0, dret = n-1;
	while(not existeix and esq <= dret)
	{
		int mig = (esq+dret)/2;
		if(taula[mig].edat == numero) existeix = true;
		else if(taula[mig].edat > numero) dret = mig-1;
		else esq = mig+1;
	}
	return existeix;
}

void ordenar_taula_per_edats(Taula_Compres taula, int n)
{
	for(int i = 0; i < n-1; i++)
        for(int j = n-1; j > i; j--)
            if(taula[j].edat < taula[j-1].edat) intercanvi(taula[j],taula[j-1]);
}

void comprovar_existencia(const Taula_Compres taula, int num, bool &existeix, int &pos, int n)
//Pre: 0 <= n <= MAX
//Post: busca si num ja esta a taula[0..n-1].numero i en cas que estigui guarda la seva posicio a pos, altrament guarda -1 a pos i false a bool
{
	existeix = false;
	pos = -1;
	int i = 0;
	while(not existeix and i < n)
	{
		if(taula[i].edat == num)
		{
			existeix = true;
			pos = i;
		}
		else i++;
	}
}

void mostrar_taula(const Taula_Compres taula, int n)
{
	for(int i = 0; i < n; i++) cout << taula[i].edat << " " << taula[i].preu << " " << taula[i].minuts << endl;
}

int transformar_a_minuts(int h_en,int h_sor)
{
    int resultat;
    int min_en = h_en % 100, min_sor = h_sor % 100;
    h_en = (h_en/100)*60;
    h_sor = (h_sor/100)*60;
    resultat = (h_sor+min_sor)-(min_en+h_en);
    return resultat;
}
void guardar_text_sumant_minuts_i_despeses(ifstream &f_entrada, Taula_Compres taula, int &n, int &minuts_totals, double &despeses)
{
	int candidat, district, pos, h_entrada, h_sortida, minut;
	f_entrada >> candidat;
	bool existeix;
	double cost;
	n = 0;
	minuts_totals = 0;
	despeses = 0;
	while(candidat != EOS and n < MAX)
	{
		comprovar_existencia(taula,candidat,existeix,pos,n);
		f_entrada >> district >> cost >> h_entrada >> h_sortida;
		minut = transformar_a_minuts(h_entrada,h_sortida);

		minuts_totals += minut;
		despeses += cost;

		if(existeix)
		{
			taula[pos].preu += cost;
			taula[pos].minuts += minut;
		}
		else
		{
			taula[n].edat = candidat;
			taula[n].preu = cost;
			taula[n].minuts = minut;
			n++;
		}
		f_entrada >> candidat;
	}
}

int main()
{
    cout.setf(ios::fixed);
    cout.precision(2);

	cout << "NOM DEL FITXER DE TEXT:" << endl;
	string nom;
	cin >> nom;
	ifstream f_entrada(nom.c_str());

	if(f_entrada.is_open())
	{
		Taula_Compres llistat;
		int n, minuts_totals;
		double despeses;
		guardar_text_sumant_minuts_i_despeses(f_entrada,llistat,n,minuts_totals,despeses);
        cout << endl;
		cout << "RESUM DE LES COMPRES REALITZADES (SEGONS ORDRE D'ENTRADA DE LES EDATS):" << endl;
		mostrar_taula(llistat,n);

        cout << endl;
		cout << "RESUM DE LES COMPRES REALITZADES (ORDENAT PER EDAT, ASCENDENT):" << endl;
		ordenar_taula_per_edats(llistat,n);
		mostrar_taula(llistat,n);

        cout << endl;
		cout << "DESPESA PER MINUT: " << despeses/minuts_totals << " EUROS/MINUT" << endl;

        cout << endl;
		cout << "ENTRAR UNA EDAT:" << endl;
		int edat;
		cin >> edat;
		bool existeix = cerca_dicotomica_booleana(llistat,n,edat);
		if(existeix) cout << "HI HA ALGU DE " << edat << " ANYS QUE HA COMPRAT" << endl;
		else cout << "NO HI HA NINGU DE " << edat << " ANYS QUE HAGI COMPRAT" << endl;
	}
	else cout << "NO S'HA TROBAT EL FITXER" << endl;

	return 0;
}
