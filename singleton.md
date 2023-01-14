# **Singleton Pattern**

## **Descrizione**

Il Singleton Pattern è un **pattern creazionale** che ha come obiettivo la **creazione di una classe che potrà avere solo una singola istanza** e che **fornisca un punto d'accesso globale a tale istanza**.

Generalmente viene utilizzato per creare oggetti di cui volgiamo essere sicuri che esista una sola istanza e per controllare l'accesso a risorse condivise.  
*Esempi: tabella dei file aperti del SO o database.*

## **Implementazione**

Tutte le implementazioni dei Singleton hanno due punti in comune:
- Il **costruttore** della classe deve essere **privato** (`private` o quantomeno `protected` se vogliamo estendere la classe) in modo da non permettere la creazione di nuove istanze con l'operatore `new`.
- Serve un **metodo creazionale statico** che faccia da costruttore. Questo internamente chiamerà il costruttore privato e salverà l'istanza ottenuta in una variabile statica, tutte le chiamate successive restituiranno sempre tale istanza senza crearne una nuova.

Vediamo un implementazione in Java:
```java
public class Singleton{

    // Costruttore privato: non permette l'uso di new
    protected Singleton(){}

    // Variabile statica: memorizza il riferimento all'unica istanza che verrà creata
    private static Singleton istanza = null;

    //Metodo creazionale statico: alla prima chiamata crea una nuova istanza e ne salva il riferimento nella variabile statica, in tutte le chiamate successive restituirà direttamente tale riferimento
    public static Singleton getIstanza(){
        if(istanza == null){
            istanza = new Singleton();
        }
        return istanza;
    }

    // Operazione eseguibile dall'unica istanza creata
    public void operazione(){ /*...*/ }

}

// Uso del singleton
public class Main{
    public static void main(String[] args){
        Singleton.getIstanza().operazione();
    }
}
```

Quest'implementazione crea l'istanza del singleton quando viene richiesta per la prima volta (**approccio lazy**), però è soggetta al **problema della concorrenza** (test e set non atomico).

Se un thread esegue il test `istanza == null` e poi il controllo passa ad un altro thread che esegue `getIstanza()`, questo creerà una nuova istanza e salverà il suo riferimento nella variabile statica. Quando il controllo tornerà al primo thread, anche lui creerà una nuova istanza.

Per risolvere questo problema basta aggiungere la keyword **`synchronized`** al metodo `getIstanza`, in modo che venga eseguito sempre in mutua esclusione. Così facendo però si utilizza un lock a livello di SO e non di programma, **riducendo enormemente le prestazioni** (inoltre il lock serve a gestire solo il primo accesso).

Volendo si può fare un doppio test innestato `istanza == null`, ma anche questa non è un implementazione accettabile in Java.

L'implementazione corretta da utilizzare attualmente è il seguente **idioma** (soluzione dipendente dal linguaggio e non dall'architettura generale):

```java
public enum Singleton{

    // Unica istanza esistente
    ISTANZA;

    // Operazione eseguibile da ISTANZA
    public void operazione(){ /*...*/ };

}

// Uso del singleton
public class Main{
    public static void main(String[] args){
        Singleton.ISTANZA.operazione();
    }
}
```

Quest'implementazione non funzionava correttamente in Java pre-5, in queste versioni veniva assegnato un indirizzo ad `ISTANZA` e poi veniva effettivamente inizializzato. Se il controllo passa ad un secondo thread che tenta di usare l'istanza non ancora inizializzata si verificano problemi.

Da Java 5 l'implementazione è stata corretta, assegnare l'indirizzo dopo l'inizializzazione la rende meno efficiente ma garantisce la correttezza.

## **Vantaggi e svantaggi**

\+ Siamo sicuri che esisterà una sola istanza accessibile globalmente  
\+ Molte caratteristiche thread safe sono già inglobate nell'implementazione fornita nativamente  
\+ Poco codice da scrivere

\- Scarsa leggibilità del codice (non è molto intuitivo)