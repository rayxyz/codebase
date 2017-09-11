# File
## Open file & read
f, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY, 0600)
if err != nil {
    panic(err)
}

defer f.Close()

if _, err = f.WriteString(text); err != nil {
    panic(err)
}
