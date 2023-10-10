# İçerik Görüşmesi Mülkat ve Cevabı(Nodejs)
1. NodeJS yada java ile https://raw.githubusercontent.com/bilalozdemir/tr-word-list/master/files/words.json buradaki listede bulunan kelimeleri tamamen random kullanarak 4gb büyüklüğünde bir txt dosyası oluştur
2. 4GB büyüklüğündeki dosyada en çok geçen 10 kelimeyi bul. Bu adımda kelimenin karakter sayısı 4'ten büyük  çok sık kullanılan bağlaçları kolayca filtrelemiş olur.

```js
const fs = require('fs');
const worldListJson = require('./wordList');

const filePath = 'new-file.txt'; // Çıktı dosyası
const totalFileSizeInBytes = 4 * 1024 * 1024 * 1024; // 4 GB
const startTime = new Date();
const writeStream = fs.createWriteStream(filePath, { flags: 'a' });
console.log('\n-> İşlem başladı...');


start();

// Dosya yoksa oluşmasını bekle
// var olun dosyanın boyutunu oku ve var olan dosyanın kalan satırlarını tamamla
async function start() {
  await controlFile()

  fs.stat(filePath, (err, stats) => {
    if (err) {
      console.error(`Hata: ${err}`);
      return;
    }

    const fileSizeInBytes = stats.size;
    function writeData() {

      const randomValue = getRandomValue(worldListJson.length)
      const wordJson = worldListJson[randomValue]
      const wordText = wordJson.word + " : " + wordJson.meanings.toString() + "\n";

      writeStream.write(wordText, (err) => {
        if (err) {
          console.error('Dosya yazma hatası:', err);
          return;
        }

        if (writeStream.bytesWritten < totalFileSizeInBytes - fileSizeInBytes) {

          // Toplam dosya boyutunu aşmadıysa bir sonraki veri bloğunu yaz
          writeData();

        } else {
          // Dosya yazma işlemi tamamlandı
          writeStream.end(() => {
            console.log(`-> Veri  ${filePath} dosyasına yazıldı.`);
            const endTime = new Date();
            const elapsedTime = endTime - startTime;

            console.log(`\n-> Yazma işlem süresi: ${Math.floor(elapsedTime / 1000 / 60 / 60)} saat ${Math.floor(elapsedTime / 1000 / 60)} dakika  ${(Math.floor(elapsedTime / 1000) % 60)} saniye. Toplam Milisaniye: ${elapsedTime}`);

            console.log("\n------------------------------\n");
            readText(elapsedTime);
          });
        }
      });
    }

    writeData();
  })
}

function readText() {
  const readline = require('readline');
  const wordMap = new Map(); // Kelimelerin sayısını tutmak için bir harita (Map)

  const rl = readline.createInterface({
    input: fs.createReadStream(filePath),
    output: process.stdout,
    terminal: false
  });

  rl.on('line', (line) => {
    // Satırdaki kelimeleri ayırır.
    const words = line.split(/\s+/);

    // Her kelimenin sayısını haritada bulur sayısını artırır.
    words.forEach((word) => {
      // Kelimenin karakter sayısı 4'ten büyükse kelimeyi ayrıştırır.
      if (word.length > 4) {
        // kelimenin sağındaki ve solundaki değerleri dikkate almaz
        const cleanedWord = word.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g, '');
        if (wordMap.has(cleanedWord)) {
          wordMap.set(cleanedWord, wordMap.get(cleanedWord) + 1);
        } else {
          wordMap.set(cleanedWord, 1);
        }
      }
    });
  });

  rl.on('close', () => {
    // Kelimeyi çoktan aza sıralar
    const sortedWords = [...wordMap.entries()].sort((a, b) => b[1] - a[1]);

    // En çok geçen 10 kelime
    const top10Words = sortedWords.slice(0, 10);

    console.log('En çok geçen 10 kelime:\n');
    top10Words.forEach((word, index) => {
      console.log(`${index + 1}. ${word[0]}: ${word[1]} kez`);
    });
    const endTime = new Date();
    const elapsedTime = endTime - startTime;

    console.log(`\n-> OkumaYazma işlem süresi: ${Math.floor(elapsedTime / 1000 / 60 / 60)} saat ${Math.floor(elapsedTime / 1000 / 60)} dakika  ${(Math.floor(elapsedTime / 1000) % 60)} saniye. Toplam Milisaniye: ${elapsedTime}`);
  });

}

console.log("\n------------------------------\n");


function getRandomValue(max) {
  return Math.floor(Math.random() * (max - 1));
}

// Rastgele değer oluştur.

function controlFile() {
  if (!fs.existsSync(filePath)) {
    console.log(`\n-> ${filePath} dosyası oluşturuldu.`);
  } else {
    console.log(`\n-> ${filePath} dosyası mevcut.`);
  }
}













```