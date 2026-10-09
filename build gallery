#!/usr/bin/env python3
"""Builds paintings.js (and fast-loading thumbnails) for the gallery.

HOW TO ADD PAINTINGS
1. Put your photos in the "paintings" folder next to this file.
   Name them like:  Les_Tournesols_1978.jpg  ->  title "Les tournesols", year 1978
2. Run:  python build_gallery.py
3. Upload this whole folder (index.html, paintings.js, paintings/, thumbs/) to your domain.

Run it again whenever you add more. Anything you edit by hand in paintings.js
(titles, notes, sizes, medium) is kept.

Recommended: pip install pillow   (creates thumbnails so 100s of paintings load fast)
"""
import json
import re
from pathlib import Path

try:
    from PIL import Image, ImageOps
except ImportError:
    Image = None
    print("Pillow not installed: skipping thumbnails. Run 'pip install pillow' for faster loading.")

ROOT = Path(__file__).parent
SRC, THUMBS, OUT = ROOT / "paintings", ROOT / "thumbs", ROOT / "paintings.js"
EXT = {".jpg", ".jpeg", ".png", ".webp"}


def load_existing():
    """Read back paintings.js so hand-edited details survive a rebuild."""
    if OUT.exists():
        m = re.search(r"=\s*(\[.*\])\s*;?\s*$", OUT.read_text(encoding="utf-8"), re.S)
        if m:
            try:
                return {p["file"]: p for p in json.loads(m.group(1))}
            except Exception:
                pass
    return {}


def parse(filename):
    """'Les_Tournesols_1978.jpg' -> ('Les tournesols', 1978)"""
    stem = re.sub(r"[_\-]+", " ", Path(filename).stem).strip()
    year = None
    m = re.search(r"\b(1[89]\d\d|20\d\d)$", stem)
    if m:
        year = int(m.group(1))
        stem = stem[: m.start()].strip()
    if not stem:
        stem = Path(filename).stem
    if not any(c.isupper() for c in stem):
        stem = stem[:1].upper() + stem[1:]
    return stem, year


def main():
    SRC.mkdir(exist_ok=True)
    THUMBS.mkdir(exist_ok=True)
    old = load_existing()
    out = []

    for f in sorted(SRC.iterdir()):
        if f.suffix.lower() not in EXT:
            continue
        rel = f"paintings/{f.name}"
        p = old.get(rel, {})
        title, year = parse(f.name)
        p.setdefault("title", title)
        if year:
            p.setdefault("year", year)
        p["file"] = rel

        if Image:
            with Image.open(f) as im:
                im = ImageOps.exif_transpose(im)
                p["w"], p["h"] = im.size
                t = THUMBS / (f.stem + ".jpg")
                if not t.exists():
                    th = im.convert("RGB")
                    th.thumbnail((900, 900))
                    th.save(t, quality=82, optimize=True)
                p["thumb"] = f"thumbs/{t.name}"
        out.append(p)

    out.sort(key=lambda p: (p.get("year") or 9999, p["title"].lower()))
    OUT.write_text(
        "window.PAINTINGS = " + json.dumps(out, ensure_ascii=False, indent=1) + ";\n",
        encoding="utf-8",
    )
    print(f"Done: {len(out)} paintings written to paintings.js")


if __name__ == "__main__":
    main()
