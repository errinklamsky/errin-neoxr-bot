const cron = require('node-cron');
const { MongoDB, PostgreSQL } = new (require('@neoxr/wb'));
const { writeFileSync: create, readFileSync: read } = require('fs');
const env = require('./config.json');
const { client } = require('./client'); // Pastikan Anda mengimpor objek client dengan benar

// Fungsi untuk melakukan pencadangan database
async function backupDatabase() {
  try {
    if (process.env.DATABASE_URL && /mongo/.test(process.env.DATABASE_URL)) {
      MongoDB.db = env.database;
    }
    const machine = (process.env.DATABASE_URL && /mongo/.test(process.env.DATABASE_URL))
      ? MongoDB
      : (process.env.DATABASE_URL && /postgres/.test(process.env.DATABASE_URL))
        ? PostgreSQL
        : new (require('./lib/system/localdb'))(env.database);

    await machine.save(global.db);
    create(`${env.database}.json`, JSON.stringify(global.db, null, 3), 'utf-8');

    // Kirim file pencadangan ke nomor pemilik
    const ownerNumber = env.owner; // Pastikan nomor pemilik terdefinisi di config.json
    await client.sendFile(ownerNumber, read(`./${env.database}.json`), `${env.database}.json`, '', {});
    console.log('Pencadangan database berhasil dan dikirim ke pemilik.');
  } catch (e) {
    console.error('Gagal melakukan pencadangan database:', e);
  }
}

// Jadwalkan tugas pencadangan setiap 12 jam
cron.schedule('0 */12 * * *', () => {
  console.log('Memulai proses pencadangan database...');
  backupDatabase();
});
