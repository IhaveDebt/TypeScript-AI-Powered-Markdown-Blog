import express, { Request, Response } from "express";
import fs from "fs/promises";
import path from "path";
import matter from "gray-matter";
import { marked } from "marked";

const app = express();
const POSTS_DIR = path.join(__dirname, "../posts");

app.get("/posts", async (req: Request, res: Response) => {
    const files = await fs.readdir(POSTS_DIR);
    const posts = await Promise.all(files.map(async (file) => {
        const content = await fs.readFile(path.join(POSTS_DIR, file), "utf-8");
        const { data } = matter(content);
        return { slug: file.replace(".md", ""), ...data };
    }));
    res.json(posts);
});

app.get("/posts/:slug", async (req: Request, res: Response) => {
    const slug = req.params.slug;
    const filePath = path.join(POSTS_DIR, `${slug}.md`);
    try {
        const content = await fs.readFile(filePath, "utf-8");
        const { data, content: md } = matter(content);
        const html = marked(md);
        res.json({ ...data, html });
    } catch {
        res.status(404).send("Post not found");
    }
});

const PORT = 5000;
app.listen(PORT, () => {
    console.log(`AI Markdown Blog running at http://localhost:${PORT}`);
});
