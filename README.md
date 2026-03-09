# Electromas.
Add file
Upload files

import ElectromasStore from "../components/ElectromasStore";

export default function Page(){
  return <ElectromasStore/>
}

'use client'
import { useEffect, useState } from "react";

const STORAGE_KEY = "electromas_products";
const ADMIN_PASSWORD = "electromas123";

export default function ElectromasStore() {

  const [products, setProducts] = useState([]);
  const [cart, setCart] = useState([]);

  const [category, setCategory] = useState("Todas");
  const [search, setSearch] = useState("");

  const [adminMode, setAdminMode] = useState(false);
  const [logged, setLogged] = useState(false);
  const [password, setPassword] = useState("");

  const [newProduct, setNewProduct] = useState({
    name: "",
    price: "",
    category: "",
    image: ""
  });

  const categories = ["Todas", "Electrodomésticos", "Cocina", "Limpieza", "Organización", "Hogar"];

  useEffect(() => {
    const saved = localStorage.getItem(STORAGE_KEY);

    if (saved) {
      setProducts(JSON.parse(saved));
    } else {
      const defaultProducts = [
        { id: 1, name: "Licuadora Profesional", price: 45000, category: "Electrodomésticos", image: "https://via.placeholder.com/400" },
        { id: 2, name: "Juego de Ollas", price: 65000, category: "Cocina", image: "https://via.placeholder.com/400" },
        { id: 3, name: "Aspiradora", price: 80000, category: "Limpieza", image: "https://via.placeholder.com/400" }
      ];

      setProducts(defaultProducts);
      localStorage.setItem(STORAGE_KEY, JSON.stringify(defaultProducts));
    }
  }, []);

  useEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(products));
  }, [products]);

  function loginAdmin() {
    if (password === ADMIN_PASSWORD) {
      setLogged(true);
      setAdminMode(true);
    }
  }

  function logoutAdmin() {
    setLogged(false);
    setAdminMode(false);
  }

  function addToCart(product) {
    const existing = cart.find((p) => p.id === product.id);

    if (existing) {
      setCart(
        cart.map((p) =>
          p.id === product.id ? { ...p, qty: p.qty + 1 } : p
        )
      );
    } else {
      setCart([...cart, { ...product, qty: 1 }]);
    }
  }

  function removeFromCart(id) {
    setCart(cart.filter((item) => item.id !== id));
  }

  function deleteProduct(id) {
    setProducts(products.filter((p) => p.id !== id));
  }

  function updatePrice(id, price) {
    setProducts(
      products.map((p) => (p.id === id ? { ...p, price: Number(price) } : p))
    );
  }

  function handleImageUpload(e) {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onloadend = () => {
      setNewProduct({ ...newProduct, image: reader.result });
    };

    reader.readAsDataURL(file);
  }

  function addProduct() {
    if (!newProduct.name || !newProduct.price) return;

    const product = {
      id: Date.now(),
      ...newProduct,
      price: Number(newProduct.price)
    };

    setProducts([...products, product]);
    setNewProduct({ name: "", price: "", category: "", image: "" });
  }

  const filteredProducts = products.filter((p) => {
    const matchCategory = category === "Todas" || p.category === category;
    const matchSearch = p.name.toLowerCase().includes(search.toLowerCase());
    return matchCategory && matchSearch;
  });

  const total = cart.reduce((sum, p) => sum + p.price * p.qty, 0);

  const whatsappMessage = encodeURIComponent(
    `Hola, quiero comprar estos productos:%0A${cart
      .map((p) => `- ${p.name} x${p.qty}`)
      .join("%0A")}%0A%0ATotal: $${total}`
  );

  const whatsappLink = `https://wa.me/5491122631692?text=${whatsappMessage}`;

  return (
    <div style={{fontFamily:"Arial",padding:20}}>

      <h1>Electromas</h1>

      <input
        placeholder="Buscar productos"
        onChange={(e)=>setSearch(e.target.value)}
      />

      <button onClick={()=>setAdminMode(!adminMode)} style={{marginLeft:10}}>
        Admin
      </button>

      {adminMode && !logged && (
        <div style={{marginTop:20}}>
          <input
            type="password"
            placeholder="Contraseña admin"
            value={password}
            onChange={(e)=>setPassword(e.target.value)}
          />
          <button onClick={loginAdmin}>Entrar</button>
        </div>
      )}

      {adminMode && logged && (
        <div style={{marginTop:20,border:"1px solid #ccc",padding:10}}>
          <h3>Agregar producto</h3>

          <input placeholder="Nombre"
            value={newProduct.name}
            onChange={(e)=>setNewProduct({...newProduct,name:e.target.value})}
          />

          <input placeholder="Precio"
            value={newProduct.price}
            onChange={(e)=>setNewProduct({...newProduct,price:e.target.value})}
          />

          <input placeholder="Categoría"
            value={newProduct.category}
            onChange={(e)=>setNewProduct({...newProduct,category:e.target.value})}
          />

          <input type="file" onChange={handleImageUpload}/>

          <button onClick={addProduct}>Agregar</button>
        </div>
      )}

      <h2 style={{marginTop:30}}>Productos</h2>

      <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(200px,1fr))",gap:20}}>

        {filteredProducts.map((p)=>(

          <div key={p.id} style={{border:"1px solid #ccc",padding:10}}>

            <img src={p.image} style={{width:"100%"}} />

            <h3>{p.name}</h3>
            <p>{p.category}</p>

            {adminMode && logged ? (
              <input
                type="number"
                value={p.price}
                onChange={(e)=>updatePrice(p.id,e.target.value)}
              />
            ):(
              <b>${p.price}</b>
            )}

            {!adminMode && (
              <button onClick={()=>addToCart(p)}>Agregar</button>
            )}

            {adminMode && logged && (
              <button onClick={()=>deleteProduct(p.id)}>Eliminar</button>
            )}

          </div>

        ))}

      </div>

      {!adminMode && (
        <div style={{marginTop:40,borderTop:"1px solid #ccc",paddingTop:20}}>
          <h2>Carrito</h2>

          {cart.map((c)=>(
            <div key={c.id}>
              {c.name} x{c.qty} - ${c.price*c.qty}
              <button onClick={()=>removeFromCart(c.id)}>Quitar</button>
            </div>
          ))}

          <h3>Total: ${total}</h3>

          <a href={whatsappLink} target="_blank">
            Comprar por WhatsApp
          </a>

        </div>
      )}

    </div>
  );
}
Commit changes
