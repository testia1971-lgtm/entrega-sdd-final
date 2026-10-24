# AUDITORIA ADVERSARIAL

## BLOQUEANTE

- **Falta de pruebas automatizadas**: los archivos de test referidos en `traceability/matrix.md` (`test_alta_persona.py`, `test_ficha_360.py`, `test_ausencias.py`, etc.) no existen en el árbol del repositorio. La regla 6 del gate (`requisito VERIFICADO sin test asociado`) falla.
  **Corrección**: crear los archivos bajo `tests/` (p. ej. `tests/test_alta_persona.py`) con al menos un test que falle si el criterio EARS no se cumple.

## IMPORTANTE

- **US‑05.* sin criterio EARS explícito**: en `specs/0006-plataforma/requirements.md` los requisitos carecen de la estructura *When …, the system shall …* requerida por la regla 1 del gate. Añadir esa línea a cada US‑05.
- **BR‑14 sigue huérfana**: aunque está declarada como gap G‑2, sigue sin referencia en ninguna spec. Añadir referencia en `specs/0006-plataforma/requirements.md` o marcar explícitamente como “gap declarado”.

## SUGERIDO

- **Documentar invariantes I‑8 e I‑9 en specs**: aparecen solo en la sección de gaps; incluir referencias en requisitos relevantes para trazabilidad completa.
- **Actualizar `designs/0003-tableros-ux.md`**: alinear comportamiento UI (filtros, zoom) con los Given/When/Then de los US correspondientes.
- **Añadir `tests/__init__.py`**: habilita reconocimiento del paquete de pruebas por Python.

---

Veredicto: **bloqueado por falta de pruebas automatizadas**.
