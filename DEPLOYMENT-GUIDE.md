# Guía de Deployment - Carritosdegolfutilitarios.com

## Información del Servidor

- **Dominio:** https://carritosdegolfutilitarios.com
- **Servidor:** 147.182.194.75
- **Usuario SSH:** forge
- **Manager:** Laravel Forge
- **Servidor Web:** Nginx

## Estructura de Directorios en Producción

```
/home/forge/carritosdegolfutilitarios.com/
├── current/                          # Symlink a la release actual
│   ├── dist/                         # Build de producción (generado por Astro)
│   ├── src/                          # Código fuente
│   ├── public/                       # Assets estáticos (imágenes, etc.)
│   └── astro.config.mjs              # Configuración de Astro
├── public.bak/                       # Backup del public anterior
└── releases/                         # Releases de Forge
```

## Configuración de Nginx

**Ubicación:** `/etc/nginx/sites-available/carritosdegolfutilitarios.com`

**Root path configurado:**
```nginx
root /home/forge/carritosdegolfutilitarios.com/current/dist;
```

**IMPORTANTE:** El root debe apuntar a la carpeta `dist` en `/current/dist`, NO a `/current/public`.

## Proceso de Deploy

### 1. Build Local

```bash
cd "C:\Desarrollo de Sofware Claude Code\carritosdegolfutilitarios\civil-cloud"
npm run build
```

Esto genera la carpeta `dist/` con los archivos estáticos optimizados.

### 2. Subir Build al Servidor

```bash
# Opción 1: Copiar carpeta dist completa
scp -r civil-cloud/dist/* forge@147.182.194.75:/home/forge/carritosdegolfutilitarios.com/current/dist/

# Opción 2: Si usas rsync (Linux/Mac)
rsync -avz --delete civil-cloud/dist/ forge@147.182.194.75:/home/forge/carritosdegolfutilitarios.com/current/dist/
```

### 3. Verificar Permisos

```bash
ssh forge@147.182.194.75 "chmod -R 755 /home/forge/carritosdegolfutilitarios.com/current/dist/"
```

### 4. Verificar Nginx (si cambió la configuración)

```bash
ssh forge@147.182.194.75 "sudo nginx -t"
ssh forge@147.182.194.75 "sudo systemctl reload nginx"
```

## Comandos Útiles

### Conectarse al Servidor
```bash
ssh forge@147.182.194.75
```

### Ver logs de Nginx
```bash
ssh forge@147.182.194.75 "sudo tail -f /var/log/nginx/error.log"
ssh forge@147.182.194.75 "sudo tail -f /var/log/nginx/access.log"
```

### Hacer Backup antes de Deploy
```bash
ssh forge@147.182.194.75 "cd /home/forge/carritosdegolfutilitarios.com/current && rm -rf dist.bak && cp -r dist dist.bak"
```

### Verificar Estado de Nginx
```bash
ssh forge@147.182.194.75 "sudo systemctl status nginx"
```

### Limpiar Cache de Nginx (si es necesario)
```bash
ssh forge@147.182.194.75 "sudo rm -rf /var/cache/nginx/*"
ssh forge@147.182.194.75 "sudo systemctl reload nginx"
```

## Configuración de Astro

**Archivo:** `civil-cloud/astro.config.mjs`

```javascript
export default defineConfig({
  site: 'https://carritosdegolfutilitarios.com',
  integrations: [mdx(), sitemap()],
  vite: {
    plugins: [tailwindcss()]
  }
});
```

**IMPORTANTE:** La URL del `site` debe coincidir con el dominio en producción.

## Estructura del Proyecto

```
carritosdegolfutilitarios/
└── civil-cloud/              # Carpeta local del proyecto
    ├── public/               # Assets estáticos (copiados tal cual a dist)
    │   ├── productos/        # Imágenes de productos (.webp)
    │   └── logo-cgu.png      # Logo principal
    ├── src/
    │   ├── pages/            # Páginas del sitio
    │   │   ├── index.astro   # Página principal
    │   │   ├── productos/
    │   │   │   └── [slug].astro  # Páginas dinámicas de productos
    │   │   └── blog/
    │   ├── components/       # Componentes reutilizables
    │   │   ├── Header.astro
    │   │   ├── Footer.astro
    │   │   └── WhatsAppButton.astro
    │   ├── layouts/
    │   │   └── BaseLayout.astro
    │   └── config/
    │       └── site.ts       # Configuración del sitio
    └── dist/                 # Build de producción (generado, no commitear)
```

**NOTA:** En el servidor, los archivos se suben directamente a `/current/` sin la carpeta `civil-cloud`.

## Productos en el Sitio

| ID | Nombre | Slug | Color |
|----|--------|------|-------|
| 3 | HAULER PRO ALUMINIUM BED BOX ELÉCTRICO | hauler-pro-aluminium-bed-box-negro | Negro |
| 13 | HAULER PRO ALUMINIUM BED BOX ELÉCTRICO | hauler-pro-aluminium-bed-box-blanco | Blanco |
| 14 | HAULER PRO FOOD SERVICE VANBOX | hauler-pro-food-service-vanbox | Blanco/Negro |

## Troubleshooting

### 404 Error en Producción

1. **Verificar que dist existe:**
   ```bash
   ssh forge@147.182.194.75 "ls -la /home/forge/carritosdegolfutilitarios.com/current/dist/"
   ```

2. **Verificar configuración de Nginx:**
   ```bash
   ssh forge@147.182.194.75 "cat /etc/nginx/sites-available/carritosdegolfutilitarios.com | grep root"
   ```
   Debe mostrar: `root /home/forge/carritosdegolfutilitarios.com/current/dist;`

3. **Verificar permisos:**
   ```bash
   ssh forge@147.182.194.75 "ls -la /home/forge/carritosdegolfutilitarios.com/current/"
   ```

4. **Reload Nginx:**
   ```bash
   ssh forge@147.182.194.75 "sudo systemctl reload nginx"
   ```

### Imágenes no Cargan

1. **Verificar que las imágenes están en dist/productos/:**
   ```bash
   ssh forge@147.182.194.75 "ls /home/forge/carritosdegolfutilitarios.com/current/dist/productos/"
   ```

2. **Las imágenes en public/ se copian automáticamente a dist/ durante el build**

### Build Falla

1. Asegurarse de tener las dependencias instaladas:
   ```bash
   npm install
   ```

2. Verificar que astro.config.mjs tenga la URL correcta

3. Limpiar caché:
   ```bash
   rm -rf .astro node_modules dist
   npm install
   npm run build
   ```

## Checklist de Deploy

- [ ] Build local exitoso (`npm run build`)
- [ ] Verificar que dist/ tiene todos los archivos
- [ ] Hacer backup en servidor (opcional pero recomendado)
- [ ] Subir dist/ al servidor vía SCP
- [ ] Verificar permisos (755)
- [ ] Verificar que Nginx apunta a la ruta correcta
- [ ] Reload Nginx si hubo cambios de configuración
- [ ] Verificar sitio en https://carritosdegolfutilitarios.com
- [ ] Commit y push de cambios a Git
- [ ] Probar todas las páginas de productos
- [ ] Verificar que las imágenes cargan correctamente

## Notas Importantes

1. **Nunca** commitear la carpeta `dist/` a Git (está en .gitignore)
2. **Siempre** hacer build local, no en el servidor
3. El servidor usa **symbolic links** (current -> releases/X), ten cuidado al modificar rutas
4. Las imágenes deben estar en formato **WebP** para mejor performance
5. El logo está en `/public/logo-cgu.png` y se copia automáticamente a `dist/`
6. Los cambios en `astro.config.mjs` requieren rebuild completo

## ⚠️ IMPORTANTE: Después de Deploy desde Forge

**Cada vez que Laravel Forge hace un deploy automático (git pull), la carpeta `dist/` desaparece porque no está en Git.**

### Pasos OBLIGATORIOS después de cada deploy de Forge:

1. **Hacer build local:**
   ```bash
   cd civil-cloud
   npm run build
   ```

2. **Subir dist al servidor:**
   ```bash
   rsync -avz --delete dist/ forge:/home/forge/carritosdegolfutilitarios.com/current/civil-cloud/dist/
   ```

3. **Recargar Nginx:**
   ```bash
   ssh forge "sudo systemctl reload nginx"
   ```

4. **Verificar que el sitio funciona:**
   - Visita: https://carritosdegolfutilitarios.com
   - Si ves 404, repite los pasos anteriores

### Script Rápido de Re-deploy:

```bash
cd /Users/soyahuehuetedigital/Documents/GitHub/carritosdegolfutilitarios/civil-cloud
npm run build && \
rsync -avz --delete dist/ forge:/home/forge/carritosdegolfutilitarios.com/current/civil-cloud/dist/ && \
ssh forge "sudo systemctl reload nginx" && \
echo "✅ Deploy completado!"
```

## Contacto del Servidor

- **Panel de Control:** Laravel Forge
- **DNS:** Configurado para apuntar a 147.182.194.75
- **SSL:** Gestionado por Forge (Let's Encrypt)

## Última Actualización

- **Fecha:** 8 de Noviembre, 2025
- **Cambios:**
  - Instalado Meta Pixel (ID: 1726876758028027) con tracking de PageView
  - Reemplazado icono genérico por logo oficial de WhatsApp en botones
  - Implementado tracking de eventos 'Contact' para clics en WhatsApp y llamadas
  - Agregada sección 'Otros productos relacionados' en páginas de producto
  - Limpieza de referencias a cushmanutilitarios.com
  - Actualización de redes sociales (solo Instagram)
- **Commits:**
  - f28dbff - Mejoras en página de productos: WhatsApp icon, Facebook Pixel y productos relacionados
  - c137099 - Limpiar referencias a cushmanutilitarios.com y duplicado de pixel
