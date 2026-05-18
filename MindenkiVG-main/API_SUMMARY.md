# API Összefoglaló és Sablonok - Vizsga Készítés

## 📋 Tartalomjegyzék
1. [API Összefoglalók](#api-összefoglalók)
2. [Közös Endpoint Sablonok](#közös-endpoint-sablonok)
3. [CRUD Műveletek Sablonok](#crud-műveletek-sablonok)
4. [Hiba Kezelés Sablonok](#hiba-kezelés-sablonok)

---

## API Összefoglalók

### 1. Books API (BooksAPI)
**Cél:** Könyv, szerző és kategória nyilvántartási rendszer
**Adatbázis:** librarydb
**Port:** 5085

#### Endpoints:
- `GET /api/books` - Összes könyv lekérdezése
- `POST /api/books?uid=FKB3F4FEA09CE43C` - Új könyv hozzáadása (UID szükséges)
- `GET /api/authors/{authorName}` - Szerző könyvei
- `GET /api/authors/Count` - Szerzők száma
- `GET /api/categories` - Összes kategória (könyvekkel)

#### Modellok:
- **Book**: Id, Cim, Szerzo, Kiadas, Kategoria
- **Author**: Id, AuthorName, Books
- **Category**: Id, Nev, Books

---

### 2. Halak API (HalakAPI)
**Cél:** Halászati nyilvántartás - halak és horgászok
**Adatbázis:** In-memory (List<T>)
**Port:** 5028

#### Endpoints:
- `GET /halak/kifogott` - Kifogott halak lekérdezése (DTO-val)
- `POST /halak` - Új hal rögzítése
- `PUT /halak` - Hal módosítása
- `DELETE /halak/{id}` - Hal törlése
- `GET /horgaszok` - Összes horgász
- `GET /horgaszok/{id}` - Horgász ID alapján

#### Modellok:
- **Hal**: Id, Nev, Faj, MeretCm, ToId, Kep
- **Horgasz**: Id, Nev, Email
- **To**: Id, Nev
- **HalakDto**: Faj, MeretCm, ToNev (Output)

---

### 3. Recept API (ReceptAPI)
**Cél:** Receptek, szakácsok és hozzávalók nyilvántartása
**Adatbázis:** receptdb
**Port:** 5287

#### Endpoints:
- `GET /api/recept/ById/{id}` - Recept lekérdezése ID-val (DTO-val)
- `GET /api/hozzavalo/All` - Összes hozzávaló
- `POST /api/hozzavalo/Uj` - Új hozzávaló
- `PUT /api/szakacs/Modosit` - Szakács módosítása
- `DELETE /api/szakacs/Torol/{id}` - Szakács törlése

#### Modellok:
- **Recept**: Id, Nev, Elkeszitesiido, NehezsegId, SzakacsId
- **Szakac**: Id, Nev, Email, Telefonszam
- **Hozzavalo**: Id, Nev, Mennyiseg
- **Nehezseg**: Id, Szint
- **ReceptDto**: Nev, ElkeszitesiIdo, NehezsegiSzint, SzakacsNev

---

### 4. Airport API (AirportAPI)
**Cél:** Repülőtéri adatok
**Adatbázis:** budairport.sql
**Port:** Nem definiált az HTTP fájlban
**Megjegyzés:** Csak adatbázis SQL fájl található, controller/endpoint információ hiányzik

---

### 5. Csatahajók API (csatahajokAPI)
**Cél:** Csatahajó nyilvántartás
**Adatbázis:** csatahajok.sql
**Port:** Nem definiált az HTTP fájlban
**Megjegyzés:** Csak adatbázis SQL fájl található, controller/endpoint információ hiányzik

---

## Közös Endpoint Sablonok

### 📌 GET - Összes elem lekérdezése
```csharp
// Controller sablon
[HttpGet]
public async Task<IActionResult> GetAll()
{
    try
    {
        var items = await _context.TableName.ToListAsync();
        return Ok(items);
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
GET http://localhost:PORT/api/[controller]
Accept: application/json
```

### 📌 GET - Elem lekérdezése ID alapján
```csharp
// Controller sablon
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    try
    {
        var item = await _context.TableName.FindAsync(id);
        if (item == null)
            return NotFound(new { message = "Elem nem található!" });
        return Ok(item);
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
GET http://localhost:PORT/api/[controller]/123
Accept: application/json
```

### 📌 GET - Szűrt lekérdezés (pl. név alapján)
```csharp
// Controller sablon
[HttpGet("{name}")]
public async Task<IActionResult> GetByName(string name)
{
    try
    {
        var item = await _context.TableName
            .FirstOrDefaultAsync(x => x.Name == name);
        if (item == null)
            return NotFound(new { message = "Nincs ilyen nevű elem!" });
        return Ok(item);
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
GET http://localhost:PORT/api/[controller]/itemname
Accept: application/json
```

### 📌 POST - Új elem hozzáadása
```csharp
// Controller sablon
[HttpPost]
public async Task<IActionResult> Add([FromBody] TableName item)
{
    if (item == null)
        return BadRequest("Üres objektum nem lehet!");
    
    try
    {
        _context.TableName.Add(item);
        await _context.SaveChangesAsync();
        return StatusCode(201, new { message = "Sikeres mentés." });
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
POST http://localhost:PORT/api/[controller]
Content-Type: application/json

{
  "field1": "value1",
  "field2": "value2"
}
```

### 📌 POST - Query paraméterrel (pl. UID ellenőrzés)
```csharp
// Controller sablon
[HttpPost]
public async Task<IActionResult> Add([FromBody] TableName item, [FromQuery] string uid)
{
    if (uid != VALID_UID)
        return Unauthorized(new { message = "Nincs jogosultsága!" });
    
    try
    {
        _context.TableName.Add(item);
        await _context.SaveChangesAsync();
        return StatusCode(201, new { message = "Sikeres mentés." });
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
POST http://localhost:PORT/api/[controller]?uid=VALID_UID
Content-Type: application/json

{
  "field1": "value1"
}
```

### 📌 PUT - Elem módosítása
```csharp
// Controller sablon
[HttpPut]
public async Task<IActionResult> Update([FromBody] TableName item)
{
    var existing = await _context.TableName.FindAsync(item.Id);
    if (existing == null)
        return NotFound(new { message = "Elem nem található!" });
    
    try
    {
        existing.Field1 = item.Field1;
        existing.Field2 = item.Field2;
        await _context.SaveChangesAsync();
        return Ok(new { message = "Sikeres módosítás." });
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
PUT http://localhost:PORT/api/[controller]
Content-Type: application/json

{
  "id": 1,
  "field1": "newvalue1",
  "field2": "newvalue2"
}
```

### 📌 PUT - Specifikus elnevezésű endpoint
```csharp
// Controller sablon
[HttpPut("ModositNev")]
public async Task<IActionResult> ModositNev([FromBody] TableName item)
{
    // ... logika
}

// HTTP kérés
PUT http://localhost:PORT/api/[controller]/ModositNev
Content-Type: application/json

{
  "id": 1,
  "field": "value"
}
```

### 📌 DELETE - Elem törlése
```csharp
// Controller sablon
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    var item = _context.TableName.Find(id);
    if (item == null)
        return NotFound(new { message = "Elem nem található!" });
    
    try
    {
        _context.TableName.Remove(item);
        _context.SaveChanges();
        return Ok(new { message = "Sikeres törlés." });
    }
    catch (Exception ex)
    {
        return BadRequest(new { message = ex.Message });
    }
}

// HTTP kérés
DELETE http://localhost:PORT/api/[controller]/123
```

### 📌 DELETE - Specifikus elnevezésű endpoint
```csharp
// Controller sablon
[HttpDelete("TorolNev/{id}")]
public IActionResult TorolNev(int id)
{
    // ... logika
}

// HTTP kérés
DELETE http://localhost:PORT/api/[controller]/TorolNev/123
```

---

## CRUD Műveletek Sablonok

### 🔄 Teljes CRUD Controller Sablon
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using YourNamespace.Models;

namespace YourNamespace.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ItemController : ControllerBase
    {
        private readonly YourDbContext _context;
        
        public ItemController(YourDbContext context)
        {
            _context = context;
        }

        // GET: api/item
        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            try
            {
                var items = await _context.Items.ToListAsync();
                return Ok(items);
            }
            catch (Exception ex)
            {
                return BadRequest(new { message = ex.Message });
            }
        }

        // GET: api/item/5
        [HttpGet("{id}")]
        public async Task<IActionResult> GetById(int id)
        {
            try
            {
                var item = await _context.Items.FindAsync(id);
                if (item == null)
                    return NotFound($"Nincs elem a megadott ID-val: {id}");
                return Ok(item);
            }
            catch (Exception ex)
            {
                return BadRequest($"Hiba: {ex.Message}");
            }
        }

        // POST: api/item
        [HttpPost]
        public async Task<IActionResult> Create([FromBody] Item item)
        {
            if (item == null)
                return BadRequest("Üres objektum nem rögzíthető!");

            try
            {
                _context.Items.Add(item);
                await _context.SaveChangesAsync();
                return StatusCode(201, "Sikeres rögzítés.");
            }
            catch (Exception ex)
            {
                return BadRequest($"Hiba: {ex.Message}");
            }
        }

        // PUT: api/item
        [HttpPut]
        public async Task<IActionResult> Update([FromBody] Item item)
        {
            var existing = await _context.Items.FindAsync(item.Id);
            if (existing == null)
                return NotFound("Nincs ilyen azonosítójú elem!");

            try
            {
                existing.Field1 = item.Field1;
                existing.Field2 = item.Field2;
                await _context.SaveChangesAsync();
                return Ok("Sikeres módosítás.");
            }
            catch (Exception ex)
            {
                return BadRequest($"Hiba: {ex.Message}");
            }
        }

        // DELETE: api/item/5
        [HttpDelete("{id}")]
        public IActionResult Delete(int id)
        {
            var item = _context.Items.Find(id);
            if (item == null)
                return NotFound("Nincs ilyen azonosítójú elem!");

            try
            {
                _context.Items.Remove(item);
                _context.SaveChanges();
                return Ok("Sikeres törlés.");
            }
            catch (Exception ex)
            {
                return BadRequest($"Hiba: {ex.Message}");
            }
        }
    }
}
```

### 📊 Include/relacionális adatok lekérdezése
```csharp
// Egy szinten kapcsolódó adatok
var item = await _context.Items
    .Include(i => i.RelatedEntity)
    .FirstOrDefaultAsync(i => i.Id == id);

// Több szinten
var item = await _context.Items
    .Include(i => i.RelatedEntity)
    .ThenInclude(r => r.AnotherRelated)
    .FirstOrDefaultAsync();

// Szűréssel és kiválasztással
var result = await _context.Items
    .Include(i => i.Category)
    .Where(i => i.IsActive)
    .Select(i => new ItemDto 
    { 
        Name = i.Name,
        CategoryName = i.Category.Name 
    })
    .ToListAsync();
```

### 📋 DTO (Data Transfer Object) minta
```csharp
// DTO osztály
public class ItemDto
{
    public string? Name { get; set; }
    public int? Value { get; set; }
    public string? CategoryName { get; set; }
}

// Konverzió a controllerben
var result = await _context.Items
    .Include(i => i.Category)
    .Select(i => new ItemDto
    {
        Name = i.Name,
        Value = i.Value,
        CategoryName = i.Category.Name
    })
    .ToListAsync();
```

---

## Hiba Kezelés Sablonok

### ✅ Try-Catch Sablon (Async)
```csharp
try
{
    var data = await _context.Items.ToListAsync();
    return Ok(data);
}
catch (Exception ex)
{
    return BadRequest(new { message = ex.Message });
}
```

### ✅ Try-Catch Sablon (Sync)
```csharp
try
{
    var data = _context.Items.ToList();
    return Ok(data);
}
catch (Exception ex)
{
    return BadRequest(new { message = ex.Message });
}
```

### ✅ Null ellenőrzés
```csharp
var item = await _context.Items.FindAsync(id);
if (item == null)
    return NotFound(new { message = "Elem nem található!" });

return Ok(item);
```

### ✅ Üres objektum ellenőrzés
```csharp
if (item == null)
    return BadRequest("Üres objektum nem rögzíthető!");
```

### ✅ Jogosultság ellenőrzés (UID/Token)
```csharp
private const string VALID_UID = "FKB3F4FEA09CE43C";

if (uid != VALID_UID)
    return Unauthorized(new { message = "Nincs jogosultsága!" });
```

### ✅ HTTP Status Kódok Referencia
```
200 OK - Sikeres GET/PUT
201 Created - Sikeres POST
400 Bad Request - Hibás kérés
401 Unauthorized - Nincs jogosultság
404 Not Found - Elem nem található
500 Internal Server Error - Szerverhiba
```

---

## 🎯 Gyakori Minták Összefoglalása

| Minta | Használat | Pél válasza |
|-------|-----------|-----------|
| **GetAll** | Összes elem | `200 OK [...]` |
| **GetById** | Egy elem | `200 OK {...}` vagy `404` |
| **Create** | Új elem | `201 Created` vagy `400` |
| **Update** | Módosítás | `200 OK` vagy `404` |
| **Delete** | Törlés | `200 OK` vagy `404` |
| **Filtered Get** | Szűrt keresés | `200 OK [...]` |
| **DTO Query** | Átalakított adat | `200 OK {...}` |

---

## 💡 Vizsga Tippek

1. **API routes**: Vigyázz az `[Route]` attribútumra - lehet `api/[controller]` vagy csak `[controller]`
2. **FromBody vs FromQuery**: `[FromBody]` POST/PUT-ben, `[FromQuery]` URL paraméterekhez
3. **Async/Await**: `ToListAsync()`, `FindAsync()` - valósdi async műveletek
4. **Include**: Relációs adatok lekérdezéséhez szükséges
5. **DTO**: Kimeneti adatformátum átalakításához
6. **SaveChanges**: Kötelező CREATE/UPDATE/DELETE után
7. **Hibakezelés**: Mindig Try-Catch - vizsgán ezt várják!

---

*Készült: Vizsga előtti felkészüléshez*
