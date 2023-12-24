```Java

public class Trie {
    private TrieNode root;

    // Trie节点定义
    private static class TrieNode {
        private TrieNode[] children; // 存储子节点的数组
        private int count; // 记录以此节点结尾的单词数量

        public TrieNode() {
            children = new TrieNode[26]; // 假设只包含小写字母
            count = 0;
        }
    }

    // 构造函数，初始化Trie树
    public Trie() {
        root = new TrieNode();
    }

    /**
     * 向Trie树中插入一个字符串。
     * 
     * @param word 待插入的字符串
     */
    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int index = word.charAt(i) - 'a';
            if (node.children[index] == null) {
                node.children[index] = new TrieNode();
            }
            node = node.children[index];
        }
        node.count++; // 标记单词结束
    }

    /**
     * 查询字符串在Trie树中出现的次数。
     * 
     * @param word 要查询的字符串
     * @return 字符串出现的次数
     */
    public int query(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int index = word.charAt(i) - 'a';
            if (node.children[index] == null) {
                return 0; // 字符串不存在
            }
            node = node.children[index];
        }
        return node.count; // 返回单词出现的次数
    }
}


```