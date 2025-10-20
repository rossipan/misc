bash
```
#!/usr/bin/env bash
# 將環境變數形式的 endpoints 轉換為 YAML 結構格式

input_file="endpoints.txt"
output_file="converted.yaml"

echo "endpoints:" > "$output_file"

while IFS=":" read -r name url; do
  # 去除多餘空白
  name=$(echo "$name" | xargs)
  url=$(echo "$url" | xargs)

  # 解析協定、主機名與埠號
  if [[ "$url" =~ ^http://([^/]+) ]]; then
    host="${BASH_REMATCH[1]}"
    port=80
  elif [[ "$url" =~ ^https://([^/]+) ]]; then
    host="${BASH_REMATCH[1]}"
    port=443
  else
    host="$url"
    port=80
  fi

  # 輸出 YAML 區塊
  cat <<EOF >> "$output_file"
  - name: $name
    url: "tcp://$host:$port"
    interval: 30s
    client:
      timeout: 1s
    conditions:
      - "[CONNECTED] == true"
EOF

done < "$input_file"

echo "✅ 轉換完成，結果已輸出到 $output_file"
```

golang
```
package main

import (
	"bufio"
	"fmt"
	"net/url"
	"os"
	"strings"
)

type Endpoint struct {
	Name       string
	URL        string
	Interval   string
	Client     struct {
		Timeout string
	}
	Conditions []string
}

func main() {
	inputFile := "endpoints.txt"
	outputFile := "converted.yaml"

	in, err := os.Open(inputFile)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error opening file: %v\n", err)
		os.Exit(1)
	}
	defer in.Close()

	out, err := os.Create(outputFile)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error creating output: %v\n", err)
		os.Exit(1)
	}
	defer out.Close()

	writer := bufio.NewWriter(out)
	defer writer.Flush()

	fmt.Fprintln(writer, "endpoints:")

	scanner := bufio.NewScanner(in)
	for scanner.Scan() {
		line := strings.TrimSpace(scanner.Text())
		if line == "" {
			continue
		}

		parts := strings.SplitN(line, ":", 2)
		if len(parts) != 2 {
			fmt.Fprintf(os.Stderr, "Skipping invalid line: %s\n", line)
			continue
		}

		name := strings.TrimSpace(parts[0])
		rawURL := strings.TrimSpace(parts[1])

		// 若缺少 scheme，加上 http:// 預設
		if !strings.HasPrefix(rawURL, "http") {
			rawURL = "http://" + rawURL
		}

		parsed, err := url.Parse(rawURL)
		if err != nil {
			fmt.Fprintf(os.Stderr, "Invalid URL for %s: %v\n", name, err)
			continue
		}

		host := parsed.Hostname()
		port := "80"
		if parsed.Scheme == "https" {
			port = "443"
		}

		fmt.Fprintf(writer, "  - name: %s\n", name)
		fmt.Fprintf(writer, "    url: \"tcp://%s:%s\"\n", host, port)
		fmt.Fprintf(writer, "    interval: 30s\n")
		fmt.Fprintf(writer, "    client:\n")
		fmt.Fprintf(writer, "      timeout: 1s\n")
		fmt.Fprintf(writer, "    conditions:\n")
		fmt.Fprintf(writer, "      - \"[CONNECTED] == true\"\n")
	}

	if err := scanner.Err(); err != nil {
		fmt.Fprintf(os.Stderr, "Error reading file: %v\n", err)
	}
	fmt.Println("✅ Conversion completed ->", outputFile)
}

```
