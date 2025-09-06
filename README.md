<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Loja Profissional de E-books</title>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
<style>
body{margin:0;font-family:'Montserrat',sans-serif;background:#121212;color:#fff}
header{text-align:center;padding:30px 20px;background:linear-gradient(90deg,#00ffcc,#00b3a0)}
header h1{margin:0;font-size:1.8em;color:#121212}
#addBookBtn{display:block;margin:15px auto;padding:10px 24px;background:#00ffcc;border:none;border-radius:8px;color:#121212;font-size:16px;cursor:pointer;transition:.3s}
#addBookBtn:hover{background:#00b3a0;color:#fff}
input[type=file]{display:none}
#categoryDropdown{display:flex;justify-content:center;margin:10px 0}
#categorySelect{padding:6px 12px;border-radius:8px;border:none;background:#1f1f1f;color:#00ffcc;font-size:14px;cursor:pointer}
#bookContainer{display:flex;overflow-x:auto;gap:15px;padding:20px;scroll-behavior:smooth}
.bookCard{min-width:160px;background:#1f1f1f;border-radius:12px;overflow:hidden;cursor:pointer;transition:.3s;display:flex;flex-direction:column;align-items:center;text-align:center;flex-shrink:0}
.bookCard:hover{transform:translateY(-5px);box-shadow:0 8px 18px rgba(0,255,204,.5)}
.bookCard img{width:100%;height:180px;object-fit:cover}
.bookCard h3{margin:6px 10px 0 10px;font-size:15px}
.bookCard p.description{font-size:12px;color:#ccc;margin:4px 10px;height:36px;overflow:hidden}
.priceTag{font-weight:bold;color:#00ffcc;margin-bottom:6px}
#pdfModal{display:none;position:fixed;z-index:999;left:0;top:0;width:100%;height:100%;background:rgba(0,0,0,.95);justify-content:center;align-items:center}
#pdfModal iframe{width:95%;height:90%;border:none;border-radius:12px}
#pdfModal span{position:absolute;top:15px;right:20px;font-size:30px;cursor:pointer;color:#00ffcc}
</style>
</head>
<body>
<header><h1>Loja Profissional de E-books</h1></header>
<button id="addBookBtn">Adicionar E-books/Símbolos</button>
<input type="file" id="bookInput" accept=".pdf,.jpg,.png" multiple>
<div id="categoryDropdown">
<select id="categorySelect"><option value="Todos">Todos</option></select>
</div>
<div id="bookContainer"></div>
<div id="pdfModal"><span id="closeModal">&times;</span><iframe id="pdfViewer"></iframe></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.12.313/pdf.min.js"></script>
<script>
const bookContainer=document.getElementById('bookContainer'),
addBookBtn=document.getElementById('addBookBtn'),
bookInput=document.getElementById('bookInput'),
pdfModal=document.getElementById('pdfModal'),
pdfViewer=document.getElementById('pdfViewer'),
closeModal=document.getElementById('closeModal'),
categorySelect=document.getElementById('categorySelect');

let books=JSON.parse(localStorage.getItem('books'))||[],
categories=JSON.parse(localStorage.getItem('categories'))||['Todos'];

function saveData(){
  localStorage.setItem('books',JSON.stringify(books));
  localStorage.setItem('categories',JSON.stringify(categories));
}

function updateCategoryDropdown(){
  categorySelect.innerHTML='';
  categories.forEach(c=>{
    const o=document.createElement('option');
    o.value=c;
    o.text=c;
    categorySelect.appendChild(o);
  })
}

addBookBtn.addEventListener('click',()=>bookInput.click());

bookInput.addEventListener('change',e=>{
  const f=e.target.files;
  for(let file of f){
    const name=prompt(`Nome do e-book "${file.name}":`,file.name),
    description=prompt(`Descrição de "${name}":`,"Descrição do e-book"),
    priceInput=prompt(`Preço de "${name}" (ex: 10,00):`,"0,00"),
    price=priceInput.replace(/[^0-9,]/g,""),
    category=prompt(`Categoria de "${name}":`,"Todos");
    if(!categories.includes(category)) categories.push(category);
    updateCategoryDropdown();
    const cover=prompt(`Link da capa (imagem) para "${name}" ou deixe vazio para gerar miniatura PDF:`,"");
    const reader=new FileReader();
    reader.onload=e=>{
      books.push({name,description,price,cover,category,type:file.type,data:e.target.result});
      saveData();
      displayBooks(categorySelect.value);
    };
    reader.readAsDataURL(file);
  }
});

async function displayBooks(filter='Todos'){
  bookContainer.innerHTML='';
  for(let book of books.filter(b=>filter==='Todos'||b.category===filter)){
    const bookCard=document.createElement('div');
    bookCard.className='bookCard';
    let imgSrc='';
    if(book.cover) imgSrc=book.cover;
    else if(book.type==='application/pdf') imgSrc=await generatePDFThumbnail(book.data);
    else imgSrc=book.data;
    bookCard.innerHTML=`<img src="${imgSrc}" alt="${book.name}"><h3>${book.name}</h3><p class="description">${book.description}</p><div class="priceTag">R$ ${book.price}</div>`;
    bookCard.addEventListener('click',()=>{
      if(book.type==='application/pdf'){
        pdfViewer.src=book.data;
        pdfModal.style.display='flex';
      }else{
        const msg=encodeURIComponent(`Olá! Tenho interesse em "${book.name}" por R$${book.price}`);
        window.open(`https://wa.me/554796872976?text=${msg}`,'_blank');
      }
    });
    bookContainer.appendChild(bookCard);
  }
}

async function generatePDFThumbnail(pdfData){
  const loadingTask=pdfjsLib.getDocument({data:atob(pdfData.split(',')[1])});
  const pdf=await loadingTask.promise;
  const page=await pdf.getPage(1);
  const viewport=page.getViewport({scale:2});
  const canvas=document.createElement('canvas'),ctx=canvas.getContext('2d');
  canvas.width=viewport.width;
  canvas.height=viewport.height;
  await page.render({canvasContext:ctx,viewport}).promise;
  return canvas.toDataURL();
}

closeModal.addEventListener('click',()=>{
  pdfModal.style.display='none';
  pdfViewer.src='';
});

categorySelect.addEventListener('change',()=>displayBooks(categorySelect.value));

updateCategoryDropdown();
displayBooks(categorySelect.value);
</script>
</body>
</html>
