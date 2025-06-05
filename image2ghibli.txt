case 'ghibli': {
 const FormData = require('form-data');
 const fs = require('fs');
 const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

 try {
 const q = m.quoted || m;
 const mime = (q.msg || q).mimetype || '';
 if (!mime.startsWith('image/')) return m.reply("❌ Kirim atau reply gambar terlebih dahulu.");

 const mediaPath = await conn.downloadAndSaveMediaMessage(q);

 const form = new FormData();
 form.append("file", fs.createReadStream(mediaPath));
 const uploadRes = await axios.post("https://cloudgood.web.id/upload.php", form, {
 headers: { ...form.getHeaders() },
 maxContentLength: Infinity,
 maxBodyLength: Infinity
 });
 if (!uploadRes.data?.url) throw new Error('❌ Upload ke CloudGood gagal');
 const imageUrl = uploadRes.data.url;

 if (fs.existsSync(mediaPath)) fs.unlinkSync(mediaPath);

 await m.reply('🌀 Proses Bosss Sabarrrrr.....');

 const bgsIdRes = await axios.get('https://apii.baguss.web.id/tools/ghibli', {
 params: {
 apikey: 'bagus',
 image: imageUrl
 }
 });

 const bgsId = bgsIdRes.data?.bgsId;
 if (!bgsId) throw new Error("❌ Gagal mendapatkan bgsId dari API.");

 let resultUrl = null;
 for (let i = 0; i < 30; i++) {
 if (i > 0) {
 const delay = (i % 3 === 0) ? 10000 : 5000;
 await sleep(delay);
 }

 try {
 const resultRes = await axios.get("https://apii.baguss.web.id/tools/ghibli/result", {
 params: {
 apikey: "bagus",
 bgsId: bgsId
 }
 });

 if (resultRes.data?.success && resultRes.data?.result?.startsWith("http")) {
 resultUrl = resultRes.data.result;
 break;
 }

 console.log(`🔁 Cek ke-${i + 1}: belum selesai.`);

 } catch (err) {
 console.log(`⚠️ Gagal cek ke-${i + 1}: ${err.message}`);
 }
 }

 if (!resultUrl) throw new Error("❌ Gagal mendapatkan hasil setelah 30x percobaan.");

 await conn.sendMessage(m.chat, {
 image: { url: resultUrl },
 caption: "✨ Success Boss!!"
 }, { quoted: m });

 } catch (e) {
 console.error(e);
 m.reply("⚠️ Terjadi kesalahan: " + e.message);
 }
}
break;