# **Iterator Pattern**

## **Descrizione**

L'Iterator Pattern è un **pattern comportamentale** che permette di **iterare sugli elmenti di una collezione senza esporne la rappresentazione interna**.

Generalmente viene utilizzato per presentare gli elementi contenuti in una collezione (array, liste, insiemi, code, alberi, ...) in maniera più pulita senza esporre informazioni riguardo l'implementazione. A volte potremmo anche voler rimuovere alcuni elementi dalla collezione durante l'iterazione.

## **Implementazione**

In generale si definisce un iteratore tramite un interfaccia che ha diversi metodi: uno per recuperare il prossimo elemento, uno per sapere se ci sono ancora elementi su cui iterare, ed eventualmente anche uno per rimuovere l'elemento corrente.  
È poi necesario definire un metodo dalla collezione che restituisca un implementazione dell'iteratore.

Questo pattern in Java viene implementato utilizzando due interfacce:

```java
public interface Iterator<T>{

    // Restituisce il prossimo elemento
    public T next();

    // Restituisce 'true' se ci sono altri elementi su cui iterare, altrimenti 'false'
    public boolean hasNext();
    
    //Rimuove l'elemento corrente dalla collezione
    public default void remove(){
        throw new UnsupportedOperationException("remove");
    }

    //Esegue un azione su ogni elemento rimanente
    public default void forEachRemaining(Consumer<? super T> action){
        Objects.requireNonNull(action);
        while(hasNext()){
            action.accept(next());
        }
    }

}
```
L'interfaccia **`Iterator<T>`** definisce l'iteratore, questa ha diversi metodi:
- `T next()`: restituisce il prossimo elemento. 
- `boolean hasNext()`: restituisce `true` se ci sono altri elementi su cui iterare, altrimenti restituisce `false`.
- `void remove()`: serve a rimuovere l'elemento attualmente puntato dall'iteratore. L'implementazione di default causa il sollevamento di una `UnsupportedOperationException`, se si vuole rendere possibile la rimozione degli elementi è necessario sovrascriverlo.
- `void forEachRemaining(Consumer<? super T>)`: è pensato per la programmazione funzionale, per ogni elemento rimanente viene eseguita una certa azione. L'implementazione di default utilizza `next` e `hasNext`.

Le sue implementazioni devono contenere al loro interno tutte le informazioni necessarie per effettuare l'iterazione (posizione corrente, algoritmo utilizzato, ...), questo permette di avere più iteratori che iterano (anche in modo diverso) sulla stessa collezione in contemporanea.

```java
public interface Iterable<T>{

    //Restituisce un implementazione di Iterator<T>
    public Iterator<T> iterator();

}
```

L'interfaccia **`Iterable<T>`** dev'essere implementata da una collezione, questa definisce un metodo `iterator` che restituisce un implementazione di `Iterator<T>`.

Implementando quest'interfaccia è possibile iterare sugli elementi utilizzando un ciclo for esteso (**foreach**), in questo modo si evita di chiamare esplicitamente i metodi `iterator`, `next` e `hasNext` rendendo così il codice più leggibile.

Ora vediamo come implementare concretamente queste interfacce:

```java
// Implementazione dell'interfaccia Iterable<T> in Multiset<T>
public class Multiset<T> implements Iterable<T>{

    // Elementi contenuti nel multiset
    private final List<T> elementi = new ArrayList<T>();

    // Il metodo 'iterator' restituisce un implementazione interna di Iterator<T>
    @Override
    public Iterator<T> iterator(){
        return new Iterator<T>(){
            // Indice dell'elemento puntato attualmente
            private int index = 0;
            // Restituisce l'elemento corrente e incrementa l'indice
            @Override
            public T next(){
                return elementi.get(index++);
            }
            // Restituisce 'true' se ci sono ancora elementi su cui iterare, altrimenti 'false'
            @Override
            public boolean hasNext(){
                return index < elementi.size();
            }
        };
    }

    // Aggiunge un elemento nel multiset
    public void add(final T elemento){
        Objects.requireNonNull(elemento);
        elementi.add(elemento);
    }

    // Altri metodi di Multiset<T>

}
```
```java
// Uso della classe Multiset<T> che implementa l'interfaccia Iterable<T>
public class Main{

    public static void main(String[] args){
        // Creiamo un nuovo multiset
        Multiset<Integer> msInt = new Multiset<>();
        // Aggiungiamo diversi elementi chiamando più volte il metodo 'add' di Multiset<T>
        msInt.add(1);
        // Avendo implementato Iterable<T> è possibile iterare usando il foreach
        for(Integer i : msInt){
            System.out.println(i);
        }
        // In alternativa possiamo iterare chiamando 'next' e 'hasNext' sull'iteratore restituito dal metodo 'iterator'
        Iterator<Integer> iter = msInt.iterator();
        while(iter.hasNext()){
            System.out.println(iter.next());
        }
    }

}
```

Molto spesso per memorizzare gli elementi di una collezione si usa un altra collezione che implementa già `Iterator<T>`, in questo caso non è necessario implementare un nuovo iteratore all'interno del metodo `iterator`, basta restituire quello già esistente.  
Nell'esempio precedente sarebbe bastato restituire `elementi.iterator()`.

Bisogna notare però che in questo caso l'iteratore già implementato permette la rimozione degli elementi, per evitare questo comportamento possiamo restituire:
- `Collections.unmodifiableCollection(elementi).iterator();`
- `new ArrayList<Integer>(elementi).iterator();`
- `List.copyOf(elementi).iterator();`

Queste soluzioni differiscono in diversi aspetti, ma in generale servono tutte a creare un iteratore che itera su una copia della collezione originale, in modo che questa non possa essere modificata.

## **Vantaggi e svantaggi**

\+ Permette di iterare sugli elementi di una collezione senza esporne l'implementazione  
\+ Si possono creare diverse implementazioni dell'iteratore che attraversano la stessa collezione utilizzando algoritmi diversi  
\+ Ci possono essere più iteratori che iterano in contemporanea sulla stessa collezione

\- In alcuni casi potrebbe risultare poco efficiente