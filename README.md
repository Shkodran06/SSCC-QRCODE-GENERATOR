  if(!img) return alert('QR nicht generiert');
  const qrSrc = img.src || img.toDataURL('image/png');
  const codeText = last4El.textContent || '';
  const printWindow = window.open('','_blank','width=1000,height=1000');
  printWindow.document.write(`
    <html>
      <head>
        <title>Druck QR</title>
        <style>
          @page{size:30mm 30mm;margin:0}
          body{margin:0;padding:0;width:30mm;height:30mm;display:flex;flex-direction:column;justify-content:center;align-items:center;font-family:Arial,sans-serif}
          img{width:24mm;height:24mm}
          .code{font-size:9pt;font-weight:bold;margin-top:1mm}
        </style>
      </head>
      <body>
        <img src="${qrSrc}" alt="QR Code">
        <div class="code">${codeText}</div>
      </body>
    </html>
  `);
  printWindow.document.close();
  printWindow.focus();
  printWindow.print();
  printWindow.close();
}

// Eventi
ssccInput.addEventListener('keydown', e=>{
  if(e.key==='Enter'){
    e.preventDefault();
    const val=ssccInput.value.trim();
    if(val){ generateQR(val); setTimeout(()=>printQR(),300); }
  }
});

clearBtn.addEventListener('click', ()=>{
  ssccInput.value='';
  generateQR('');
});

printTestBtn.addEventListener('click', ()=>printQR());

downloadBtn.addEventListener('click', ()=>{
  if(!qrCode) return alert('QR nicht generiert');
  const img=qrContainer.querySelector('img')||qrContainer.querySelector('canvas');
  if(!img) return alert('QR-Bild nicht verfügbar');
  const link=document.createElement('a');
  link.href=img.src;
  link.download='SSCC_'+ssccInput.value+'.png';
  link.click();
});
</script>

</body>
</html>
