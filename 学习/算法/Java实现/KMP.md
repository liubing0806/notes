```Java

public class KMP {

    /**
     * 构建KMP算法中的next数组。
     * next数组存储在每个位置上，当模式串与文本串不匹配时，模式串应该回退的位置。
     * 
     * @param pattern 模式串
     * @return next数组
     */
    private static int[] buildNext(String pattern) {
        int m = pattern.length();
        int[] next = new int[m];
        for (int i = 1, j = 0; i < m; i++) {
            // 当前字符不匹配时，回退到next[j - 1]
            while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) {
                j = next[j - 1];
            }
            // 当前字符匹配，增加j的长度
            if (pattern.charAt(i) == pattern.charAt(j)) {
                j++;
            }
            next[i] = j;
        }
        return next;
    }

    /**
     * 使用KMP算法在文本中搜索模式串。
     * 
     * @param text    文本串
     * @param pattern 模式串
     * @return 如果模式串在文本中出现返回true，否则返回false
     */
    public static boolean kmpSearch(String text, String pattern) {
        int[] next = buildNext(pattern);
        for (int i = 0, j = 0; i < text.length(); i++) {
            // 当前字符不匹配时，回退到next[j - 1]
            while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
                j = next[j - 1];
            }
            // 当前字符匹配，增加j的长度
            if (text.charAt(i) == pattern.charAt(j)) {
                // 如果整个模式串都匹配了，返回true
                if (j == pattern.length() - 1) {
                    return true; // 匹配成功
                }
                j++;
            }
        }
        return false; // 匹配失败
    }
}


```