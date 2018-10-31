#  <center>Spatiotemporal Analysis in Spark & Hadoop</center>

# <center>  Εισαγωγή</center>

Σε αυτή την έρευνα παρουσιάζεται ένα σύστημα για την παρακολούθηση της θαλάσσιας δραστηριότητας βάσει των θέσεων των πλοίων που πλέουν στη θάλασσα. Γενικά η ανίχνευση τέτοιων δεδομένων γίνεται και επικεντρώνεται στις σημαντικές αλλαγές πορείας του κάθε σκάφους στο πέρασμα του χρόνου και έτσι μπορούμε να κρατήσουμε ιστορικά δεδομένα για την πρόσφατη κίνηση των πλοίων. Επιπλέον, χάρη στην αναγνώριση σύνθετων γεγονότων, αυτό το σύστημα μπορεί επίσης να προσφέρει άμεση ειδοποίηση σε περίπτωση θαλάσσιας κατάστασης έκτακτης ανάγκης, όπως ο κίνδυνος των συγκρούσεων, ύποπτες κινήσεις σε προστατευόμενες ζώνες, ή ανταλλαγή παράνομων φορτίων στην ανοιχτή θάλασσα. Τα δεδομένα τα οποία θα ασχοληθούμε είναι χωροχρονικά όπως αναφέραμε αλλά και γεωγραφικά. Η ερεύνα μας θα εστιάσει στην παρακολούθηση αλιευτικών πλοίων που βρίσκονται εντος απαγορευμένων αλιευτικών χώρων. Θα γίνει μια ανάλυση χρονολογικής σειράς, η οποία θα μελετά την συχνότητα των πλοίων που είναι εντος αυτής στο βάθος του χρόνου με το μοντέλο AR( autoregressive ). Τέλος θα εκτελέσουμε τον αλγόριθμο ομαδοποίησης K-means για τα αλιεύτηκα πλοία εντος των επιτρεπομένων περιοχών αλίευσης. Για τον λόγο ότι ο όγκος των δεδομένων που έχουμε είναι τεράστιος και δύσκολα επεξεργάσιμος χρησιμοποιήσαμε εργαλεία οπού προσφέρονται για την ανάλυση μεγάλης κλίμακας δεδομένων όπως είναι το Hdfs για την παράλληλη αποθήκευση των δεδομένων σε cluster υπολογιστών καθώς και το Spark για την γρήγορη επεξεργασίας μεγάλης κλίμακας παράλληλων δεδομένων.

# <center> Περιγραφή Δεδομένων </center>

Τα δεδομένα τα όποια έχουμε χρησιμοποιήσει παρέχονται από την ιστοσελίδα του zenodo ( Heterogeneous Integrated Dataset for Maritime Intelligence, Surveillance, and Reconnaissance ) και χρονικά αναφέρονται για το εξάμηνο 2015-10-01 έως 2016-03-31. Η συγκεκριμένη ιστοσελίδα είχε πολλά data sets τα όποια αφορούν δεδομένα κινούμενων πλοίων, περιβάλλοντος (μετεωρολογικά), γεωγραφικά δεδομένα καθώς και άλλα. Από όλα αυτά τα σύνολα δεδομένων στην παρούσα εργασία θα χρησιμοποιήσουμε τα εξής :
<ol>
<li> [P1] AIS Data (nari_dynamic .csv, nari_static.csv ) </li>
<li> [P1] AIS Status Codes and Types</li>
<li>[C5] Fishing Constraints</li>
<li> [C4] Fishing Areas (European commission)</li>
</ol>

<h1></h1>
1.Για το τα δεδομένα [P1] AIS Data - nari_dynamic έχουμε

<ul>
<li>Sourcemmsi : Είναι το μοναδικό id κάθε πλοίου</li>
<li>Navigationalstatus : Ακέραιος αριθμός που αναφέρετε στην κατάσταση του πλοίου</li>
<li>Speedoverground : Ταχύτητα του πλοίου σε κόμβους</li>
<li>Lon : Είναι το γεωγραφικό μήκος</li>
<li>Lat : Είναι το γεωγραφικό πλάτος</li>
<li>t : χρόνος σε epoch/unix timestamp</li></ul>

<h1></h1>

2. Για το τα δεδομένα [P1] AIS Data - nari_static έχουμε
<ul>
<li>Sourcemmsi : Το μοναδικό id κάθε πλοίου </li>
<li>shipname : Το όνομα του πλοίου</li>
<li>shiptype : Ο τύπος του πλοίου </li>
<li>t : χρόνος σε epoch/unix timestamp</li> </ul>

<h1></h1>

3. Για το τα δεδομένα [P1] AIS Status Codes and Types έχουμε
<ul>
<li>Shiptype(min,max) : Ο αριθμός που αντιστοιχεί στον τύπο του πλοίου.</li> 
<li>Ais_type_summary : Η ονομασία του τύπο των πλοίων.</li> </ul>
 
 <h1></h1>
 
4. Για το τα δεδομένα [C5] Fishing Constraints έχουμε
<ul>
<li>Geometry : Περιέχει τα πολύγωνα ‘’points’’ των 2 περιοχών</li>
<li>Geoid : Περιέχει την αρίθμηση των περιοχών δηλαδή πρώτη περιοχή=0 και η δεύτερη = 1</li> </ul>

<h1></h1>

5. Για το τα δεδομένα [C4] Fishing Areas (European commission) έχουμε
<ul>
<li>Name : Το όνομα του κάθε πολυγώνου (Σε όλα είναι BREST)</li> 
<li>maxLat, maxLong, minLat, minLong : Είναι τα bounds του ενιαίου πολυγώνου</li> 
<li>geometry : Περιέχει τα πολύγωνα ‘’points’’ των 2 περιοχών</li> 
<li>geoid : Περιέχει την αρίθμηση των περιοχών</li> </ul>

<h1></h1>

# <center>  Περιγραφή Εργαλείων Που Χρησιμοποιηθήκαν </center>
Ο Μεγάλος όγκος δεδομένων προκάλεσε τεράστια αλλαγή στα εργαλεία όπου χρησιμοποιούνται για την αποθήκευση και την ανάλυση δεδομένων. Μια γλωσσά προγραμματισμού εγκατεστημένη σε έναν υπολογιστή δεν είναι ικανή να τα φέρει εις πέρας σε ένα Big Data πρόβλημα. Για την επίλυση αυτού το προβλήματος έχει δημιουργηθεί το Spark για την επεξεργασία των δεδομένων , καθώς και το HDFS του Hadoop για την αποθήκευση . Στην συγκεκριμένη εργασία χρησιμοποιήσαμε και τα δυο αυτά εργαλεία για την υλοποίηση της ερευνά που έγινε ως προς την παράβαση αλιευτικών πλοίων εντος απαγορευμένων περιέχων. Παρακάτω γίνεται μια μικρή επισκόπηση για τα δυο αυτά εργαλεία.

# 1. Εργαλείο Επεξεργασίας δεδομένων Spark

Το Apache Spark δημιουργήθηκε με την γλώσσα προγραμματισμού Scala άλλα έχει την δυνατότητα να χρησιμοποιηθεί και με τις γλώσσες (Python, R, Java). Είναι μία γρήγορη και γενικής χρήσεως μηχανή επεξεργασίας μεγάλης κλίμακας παράλληλων δεδομένων, η οποία επεκτείνει το δημοφιλές μοντέλο MapReduce. Μπορούμε να πούμε ότι ένα από τα πιο σημαντικά χαρακτηριστικά του Spark είναι ότι λειτουργεί στην μνήμη [1]. Το γεγονός αυτό, έχει ως αποτέλεσμα να είναι πιο αποτελεσματικό σε υπολογισμούς, λόγω της αποφυγής του δίσκου ανάγνωσης / εγγραφής συμφόρησης. Το έργο Spark περιέχει πολλές συνιστώσες: Spark SQL, Spark Streaming, MlLib και GraphX. Αυτές οι συνιστώσες έχουν σχεδιαστεί για να εργαστούν από κοινού, όποτε το θελήσει κανείς. Έτσι, μπορούν να συνδυαστούν σε ένα έργο λογισμικού, όπου το Spark αποτελεί τον πυρήνα τους. Τότε το SPARK είναι υπεύθυνο για τον προγραμματισμό, τη διανομή, και τις εφαρμογές παρακολούθησης ενός συμπλέγματος (cluster). Ένα από τα πιο σημαντικά εργαλεία στην Spark είναι η βιβλιοθήκη μηχανικής μάθησης . Αυτή η βιβλιοθήκη είναι μέρος του πυρήνα Spark, ως εκ τούτου μπορεί να χρησιμοποιηθεί από οποιαδήποτε άλλη από τις άλλες βιβλιοθήκες. Ο σχεδιασμός και η φιλοσοφία Mllib είναι απλή δηλαδή επιτρέπει να επικαλούνται διάφοροι αλγόριθμοι σε κατανεμημένα σύνολα δεδομένων, που αντιπροσωπεύουν όλα τα δεδομένα σε RDDs. Υλοποιήσαμε έναν αλγόριθμο (K-means) με την χρήση της βιβλιοθήκης Mllib.

# 2. Hadoop Distributed File System (HDFS)

Το σύστημα κατανομής αρχείων Hadoop (HDFS) είναι ένα κατανεμημένο σύστημα αρχείων που έχει σχεδιαστεί για να τρέχει σε όλα τα υπάρχοντα συστήματα Hardware [2]. Έχει πολλές ομοιότητες με τα υπάρχοντα κατανεμημένα συστήματα αρχείων. Ωστόσο, οι διαφορές από άλλα κατανεμημένα συστήματα αρχείων είναι σημαντικές. Το HDFS είναι εξαιρετικά ανεκτικό σε σφάλματα και έχει σχεδιαστεί για ανάπτυξη σε υλικό χαμηλού κόστους. Το HDFS παρέχει υψηλή πρόσβαση σε δεδομένα εφαρμογών και είναι κατάλληλο για εφαρμογές που έχουν μεγάλα σύνολα δεδομένων. Επίσης χαλαρώνει μερικές απαιτήσεις POSIX για να επιτρέψει τη ροή δεδομένων σε δεδομένα αρχείων. Το HDFS δημιουργήθηκε αρχικά ως υποδομή για το πρόγραμμα μηχανών αναζήτησης του Apache Nutch και είναι μέρος του Apache Hadoop Core.

# 3. Γλωσσά προγραμματισμού Python - Pyspark

Ο προγραμματισμός των σύγχρονων συστημάτων μηχανικής μάθησης και cluster computing γίνεται συνήθως με Python. Ακόμη και όταν η Python δεν είναι η κύρια γλώσσα προγραμματισμού ενός τέτοιου συστήματος, υπάρχει ένα ‘Python binding’. Χαρακτηριστικό παράδειγμα είναι το Spark που, αν και το πρωτεύον API του είναι σε Scala, κυρίως χρησιμοποιείται το Python API (PySpark). Η Python είναι επίσης η γλώσσα που χρησιμοποιούν πλέον οι περισσότεροι data scientists λόγω των εκτεταμένων βιβλιοθηκών έτοιμων αλγορίθμων μηχανικής μάθησης και επεξεργασίας δεδομένων διαφορετικών μορφών και μεγεθών.

# 4. FileZilla Client

Το πρόγραμμα FileZilla Client δίνει την δυνατότητα για την μεταφορά αρχείων από έναν υπολογιστή A σε έναν άλλο μακρινό υπολογιστή B. Το συγκεκριμένο εργαλείο χρησιμοποιήθηκε για την μεταφορά όλων μας των δεδομένων που είχαμε κατεβάσει από το zenodo στο vm Master όπου είχαμε φτιάξει στον okeano

# <center> Περιγραφή της αρχιτεκτονικής </center> 

# Αρχιτεκτονική του Spark

Το Spark χρησιμοποιεί αρχιτεκτονική master/worker. Υπάρχει ένας driver που μιλάει με έναν συντονιστή που ονομάζεται master και διαχειρίζεται τους workers στους οποίους τρέχουν οι executors. Ένας master είναι ένα Spark instance που συνδέεται με έναν Cluster Manager για πόρους και αποκτά κόμβους της συστάδας για να εκτελέσει executors. Από την άλλη πλευρά, οι workers (γνωστοί και ως slaves) αποτελούν Spark instances στα οποία οι executors ζουν για να εκτελούν εργασίες. Αυτοί είναι οι υπολογιστικοί κόμβοι του Spark που επίσης επικοινωνούν μεταξύ τους χρησιμοποιώντας τα Block Manager instances που διαθέτουν [3]. Ο driver και οι executors τώρα, τρέχουν στις δικές τους Python διαδικασίες. Μπορούν να τρέξουν όλοι στην ίδια (οριζόντια συστάδα) ή σε ξεχωριστές μηχανές (κάθετη συστάδα) ή σε μικτή διαμόρφωση μηχανής. Όταν δημιουργείται ένα SparkContext, κάθε worker εκκινεί έναν executor. Οι executors συνδέονται πίσω στο πρόγραμμα του driver. Τώρα ο driver μπορεί να τους στείλει εντολές, όπως για παράδειγμα flatMap, map και reduceByKey. Όταν ο driver κλείσει, οι executors κλείνουν επίσης. Υπάρχουν τρείς διαφορετικοί τύποι cluster manager στο Apache Spark:
1. Spark-Standalone – Spark workers are registered with spark master
2. Yarn – Spark workers are registered with YARN Cluster manager.
3. Mesos – Spark workers are registered with Mesos.
Στην συγκεκριμένη εργασία θα χρησιμοποιήσαμε ως cluster manager τον δεύτερο, δηλαδή το Yarn-SPARK. Με σκοπό την υλοποίηση της εργασίας μας δημιουργήσαμε δυο VMs στον okeano με τα έξης χαρακτηριστικά :
<ul>
<li>Ubuntu Lts</li>
<li>Cpu : x8</li>
<li>Ram : 8</li>
<li>Private network</li>
<li>IPv6</li>
<li>Storage Hdd : 40 Gb</li>
</ul?>
Έπειτα κατεβάσαμε την 2.3.1 έκδοση του Spark και κάναμε τις ανάλογες ρυθμίσεις με βάση το Guide που μας δόθηκε. Τέλος ορίσαμε τον master και τους workers (slaves).
<h1></h1>
Image : Spark Architecture
<h1></h1>
<img src="http://datastrophic.io/content/images/2016/03/Spark-Cluster-Architecture--1-.png" alt="Spark Architecture" class="center">
<h1></h1>
<h1></h1>
# Αρχιτεκτονική του Hadoop

Το HDFS είναι ένα σύστημα σχεδιασμένο για την αποθήκευση πολύ μεγάλου όγκου δεδομένων, με υποστήριξη για πρότυπα προσπέλασης σε δεδομένα συνεχούς ροής (streaming data access patterns), σε cluster απλών υπολογιστών. Υπάρχουν σήμερα συστάδες Hadoop σε λειτουργία, που αποθηκεύουν Petabytes δεδομένων. Το HDFS είναι χτισμένο γύρω από την ιδέα ότι το πιο αποτελεσματική πρότυπο επεξεργασίας δεδομένων είναι αυτό της «εγγραφής-μια φορά, ανάγνωσης-πολλές φορές» (write-once, read-multiple times). Τυπικά, ένα σύνολο δεδομένων παράγεται ή αντιγράφεται από προϋπάρχουσα πηγή, και στη συνέχεια πραγματοποιούνται διάφορες αναλύσεις σε αυτό το σύνολο δεδομένων. Κάθε ανάλυση περιλαμβάνει ένα μεγάλο μέρος, αν όχι όλο, από το σύνολο των δεδομένων. Κατά συνέπεια, ο χρόνος για την ανάγνωση ολόκληρου του συνόλου δεδομένων είναι πιο σημαντικός από τον χρόνο ανάγνωσης της πρώτης εγγραφής. Στην δική μας περίπτωση έχουμε 2 VM μηχανήματα σε κάθε cluster τα οποία βρίσκονται στον cloud του okeanos . Είναι δυο Ubuntu LTS τα οποία βρίσκονται σε private network με ΝΑΤ, σύμφωνα με τo Guide για το ένα cluster κα για το δεύτερο cluster απλά με public IPs.Στον master τρέχει ο NameNode , o Secondary NameNode, o NodeManager , o ResourceManager και έχουμε βάλει και εκεί να τρέχει και ένας DataNode. Ενώ στον slave τρέχει μόνο ο NodeManager και ο Datanode.
<h1></h1>
Image : Hdfs Architecture
<h1></h1>
<img src="https://www.mssqltips.com/tipimages2/3180_HDFS_Architecture.jpg" alt="Hdfs Architecture" class="center">
<h1></h1>
<h1></h1>
