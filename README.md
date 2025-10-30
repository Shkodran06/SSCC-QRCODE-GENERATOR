<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SSCC QR-CODE GENERATOR</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: #1e1e1e;
  color: #f0f0f0;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  max-width: 900px;
  width: 90%;
  background: #2b2b2b;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 3px 15px rgba(0,0,0,0.4);
  justify-content: center;
  align-items: flex-start;
}

.controls {
  flex: 1 1 280px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.input-row {
  display: flex;
  gap: 8px;
  align-items: center;
}

.input-row input {
  flex: 1;
  padding: 10px;
  border-radius: 6px;
  border: none;
  font-size: 1rem;
}

/* X completamente rossa */
.input-row button {
  flex: 0 0 auto;
  padding: 10px;
  font-size: 1.4rem;
  background: #ff0000;
  color: white;
  border-radius: 6px;
  border: none;
  cursor: pointer;
}

.input-row button:hover { background: #cc0000; }

.controls button {
  padding: 8px 12px;
  border-radius: 6px;
  border: none;
  background: #0b5cff;
  color: white;
  cursor: pointer;
  font-size: 0.95rem;
}

.controls button:hover { background: #0736d1; }

.history-title {
  margin-top: 15px;
  font-size: 0.95rem;
  font-weight: bold;
}

.history {
  max-height: 150px;
  overflow-y: auto;
  font-size: 0.9rem;
}

.history-item {
  padding: 4px 0;
  border-bottom: 1px solid #444;
  cursor: pointer;
  user-select: text; /* permette di selezionare e copiare */
}

.history-item:hover { text-decoration: underline; }

.preview-card {
  flex: 1 1 260px;
  background: #f8f8f8;
  color: #000;
  border-radius: 10px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

#qrContainer {
  width: 180px;
  height: 180px;
  background: #fff;
  border-radius: 8px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 5px;
}
