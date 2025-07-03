# Grafos-taller2
#include <iostream>
#include <fstream>

using namespace std;

const int INFINITO = 999999;

struct NodoArista {
    int destino;
    int peso;
    NodoArista* siguiente;
    
    NodoArista(int d, int p) : destino(d), peso(p), siguiente(nullptr) {}
};

class Grafo {
    int numVertices;
    NodoArista** listaAdy;
    
public:
    Grafo(int V) : numVertices(V) {
        listaAdy = new NodoArista*[V];
        for (int i = 0; i < V; ++i)
            listaAdy[i] = nullptr;
    }
    
    ~Grafo() {
        for (int i = 0; i < numVertices; ++i) {
            NodoArista* actual = listaAdy[i];
            while (actual != nullptr) {
                NodoArista* temp = actual;
                actual = actual->siguiente;
                delete temp;
            }
        }
        delete[] listaAdy;
    }
    
    // Método público para obtener el número de vértices
    int obtenerNumVertices() const {
        return numVertices;
    }
    
    void agregarArista(int origen, int destino, int peso) {
        NodoArista* nuevoNodo = new NodoArista(destino, peso);
        nuevoNodo->siguiente = listaAdy[origen];
        listaAdy[origen] = nuevoNodo;
        
        nuevoNodo = new NodoArista(origen, peso);
        nuevoNodo->siguiente = listaAdy[destino];
        listaAdy[destino] = nuevoNodo;
    }
    
    void prim() {
        int* padre = new int[numVertices];
        int* clave = new int[numVertices];
        bool* enprim = new bool[numVertices];
        
        for (int i = 0; i < numVertices; ++i) {
            clave[i] = INFINITO;
            enprim[i] = false;
        }
        
        clave[0] = 0;
        padre[0] = -1;
        
        for (int contador = 0; contador < numVertices - 1; ++contador) {
            int u = -1;
            int minClave = INFINITO;
            for (int v = 0; v < numVertices; ++v) {
                if (!enprim[v] && clave[v] < minClave) {
                    minClave = clave[v];
                    u = v;
                }
            }
            
            if (u == -1) break;
            
            enprim[u] = true;
            
            NodoArista* actual = listaAdy[u];
            while (actual != nullptr) {
                int v = actual->destino;
                int peso = actual->peso;
                if (!enprim[v] && peso < clave[v]) {
                    padre[v] = u;
                    clave[v] = peso;
                }
                actual = actual->siguiente;
            }
        }
        
        cout << "\n=== ARBOL MINIMO COBERTOR (PRIM) ===\n";
        cout << "Arista \tPeso\n";
        for (int i = 1; i < numVertices; ++i)
            cout << padre[i] << " - " << i << "\t" << clave[i] << endl;
        
        delete[] padre;
        delete[] clave;
        delete[] enprim;
    }
    
    void dijsktra(int origen, int destino) {
        int* distancia = new int[numVertices];
        int* padre = new int[numVertices];
        bool* visitado = new bool[numVertices];
        
        for (int i = 0; i < numVertices; ++i) {
            distancia[i] = INFINITO;
            visitado[i] = false;
            padre[i] = -1;
        }
        
        distancia[origen] = 0;
        
        for (int contador = 0; contador < numVertices - 1; ++contador) {
            int u = -1;
            int minDist = INFINITO;
            for (int v = 0; v < numVertices; ++v) {
                if (!visitado[v] && distancia[v] < minDist) {
                    minDist = distancia[v];
                    u = v;
                }
            }
            
            if (u == -1 || u == destino) break;
            
            visitado[u] = true;
            
            NodoArista* actual = listaAdy[u];
            while (actual != nullptr) {
                int v = actual->destino;
                int peso = actual->peso;
                if (!visitado[v] && distancia[u] + peso < distancia[v]) {
                    distancia[v] = distancia[u] + peso;
                    padre[v] = u;
                }
                actual = actual->siguiente;
            }
        }
        
        cout << "\n=== CAMINO MINIMO (DIJKSTRA) ===\n";
        cout << "Desde " << origen << " hasta " << destino << ":\n";
        
        if (distancia[destino] == INFINITO) {
            cout << "No existe camino entre los vertices\n";
        } else {
            int* camino = new int[numVertices];
            int indice = 0;
            for (int v = destino; v != -1; v = padre[v])
                camino[indice++] = v;
            
            cout << "Camino: ";
            for (int i = indice - 1; i >= 0; --i) {
                if (i < indice - 1) cout << " -> ";
                cout << camino[i];
            }
            cout << "\nDistancia total: " << distancia[destino] << endl;
            
            delete[] camino;
        }
        
        delete[] distancia;
        delete[] padre;
        delete[] visitado;
    }
    
    void colorearGrafo() {
        int* color = new int[numVertices];
        bool* disponible = new bool[numVertices];
        
        for (int i = 0; i < numVertices; ++i) {
            color[i] = -1;
            disponible[i] = true;
        }
        
        color[0] = 0;
        
        for (int u = 1; u < numVertices; ++u) {
            NodoArista* actual = listaAdy[u];
            while (actual != nullptr) {
                int v = actual->destino;
                if (color[v] != -1)
                    disponible[color[v]] = false;
                actual = actual->siguiente;
            }
            
            int cr;
            for (cr = 0; cr < numVertices; ++cr)
                if (disponible[cr]) break;
            
            color[u] = cr;
            
            for (int i = 0; i < numVertices; ++i)
                disponible[i] = true;
        }
        
        int numCromatico = 0;
        for (int u = 0; u < numVertices; ++u)
            if (color[u] + 1 > numCromatico)
                numCromatico = color[u] + 1;
        
        cout << "\n=== COLORACION DE GRAFOS ===\n";
        cout << "Asignacion de colores:\n";
        for (int u = 0; u < numVertices; ++u)
            cout << "Vertice " << u << ": Color " << color[u] << endl;
        cout << "Numero cromatico: " << numCromatico << endl;
        
        delete[] color;
        delete[] disponible;
    }
};

Grafo leerArchivo(const char* nombreArchivo) {
    ifstream archivo(nombreArchivo);
    if (!archivo) {
        cerr << "Error al abrir el archivo " << nombreArchivo << endl;
        exit(1);
    }
    
    int numVertices;
    archivo >> numVertices;
    Grafo g(numVertices);
    
    int u, v, w;
    while (archivo >> u >> v >> w) {
        g.agregarArista(u, v, w);
    }
    
    archivo.close();
    return g;
}

void mostrarMenu() {
    cout << "\n=== MENU PRINCIPAL ===";
    cout << "\n1. Mostrar Arbol Minimo Cobertor (Prim)";
    cout << "\n2. Buscar Camino Minimo (Dijkstra)";
    cout << "\n3. Mostrar Coloracion del Grafo";
    cout << "\n4. Salir";
    cout << "\nSeleccione una opcion (1-4): ";
}

int main() {
    Grafo g = leerArchivo("taller2-ejemplo.txt");
    
    int opcion;
    do {
        mostrarMenu();
        cin >> opcion;
        
        switch(opcion) {
            case 1:
                g.prim();
                break;
            case 2: {
                int origen, destino;
                cout << "Ingrese vertice origen (0-" << g.obtenerNumVertices()-1 << "): ";
                cin >> origen;
                cout << "Ingrese vertice destino (0-" << g.obtenerNumVertices()-1 << "): ";
                cin >> destino;
                g.dijsktra(origen, destino);
                break;
            }
            case 3:
                g.colorearGrafo();
                break;
            case 4:
                cout << "Saliendo del programa...\n";
                break;
            default:
                cout << "Opcion no valida. Intente nuevamente.\n";
        }
    } while (opcion != 4);
    
    return 0;
}
