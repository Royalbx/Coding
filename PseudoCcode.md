class SARDataProcessor:
    """  构造一个数据处理类  """
    def __init__(self, image_folder):
        self.image_folder = image_folder
        self.image_num = len(os.listdir(image_folder))
        """应从头文件中获取矩阵宽高"""
        self.height
        self.width  
    
    def parsing(self, path, height, width):
        """将二进制文件进行解析，返回实部图和虚部"""
        """"""
        file = open(path, 'rb')
        初始化一个形状为(height, 2 * width)的空的array pic
        for i in range(height):
            for j in range(2*width):
                data = file.read(4)
                elem = struct.unpack("f", data)[0]
                pic[i][j] = elem
        file.close()

        # 提取实部数据和虚部数据
        real = pic[:, ::2]
        imag = pic[:, 1::2]

        return real, imag

    def average_amplitude(self, folder_path, save_path):
        """ 计算平均幅度值方法 """
        """ 
        逐一读入影像的二进制文件 
        将二进制文件解析为实部和虚部，再组合为复数数据
        计算单张影像振幅
        平均振幅 = sum（单张影像振幅/图像数量）
        """
        height = self.height, width = self.width
        A_amplitude = np.zeros(height, width)
        for image in os.listdir(folder_path):
            real, imag = self.parsing(os.path.join(folder_path, image), height, width)
            complex_data = real + 1j * imag
            A_amplitude += np.abs(complex_data)/self.image_num
        """将平均幅度转为图像对象并保存"""
        JPEG = Image.fromarray(A_amplitude)
        JPEG.save(save_path)

        return JPEG

    def amplitude_nsd(self, folder_path, save_path):
        """ 计算幅度归一化标准差方法 """
        
        height = self.height
        width = self.width
        A_amplitude = np.zeros((height, width))
        # 创建一个临时文件夹，将每幅影像的振幅暂时是存入磁盘，需要时再从磁盘读取，避免内存占用
        with tempfile.TemporaryDirectory() as temp_dir:
            # 遍历文件夹中的每个图像文件
            for image in os.listdir(folder_path):
                real, imag = self.parsing(os.path.join(folder_path, image), height, width)
                complex_data = real + 1j * imag
                amplitude = np.abs(complex_data)

                # 将各幅影像的幅度图保存到临时文件夹中
                np.save(os.path.join(temp_dir, image + '_amplitude.npy'), amplitude)

                # 更新A_amplitude
                A_amplitude += amplitude / self.image_num

            # 逐一读取临时存储的幅度图并计算方差
            variance = np.zeros((height, width))
            for temp_amplitude_file in os.listdir(temp_dir):
                temp_amplitude_path = os.path.join(temp_dir, temp_amplitude_file)
                amplitude = np.load(temp_amplitude_path)
                variance += ((amplitude - A_amplitude) ** 2) / self.image_num

                # 删除读取的临时幅度图文件
                os.remove(temp_amplitude_path)

        std =  np.sqrt(variance)
        amplitude_nsd = std / A_amplitude

        """将平均幅度转为图像对象并保存"""
        JPEG = Image.fromarray(amplitude_nsd)
        JPEG.save(save_path)
        return JPEG



if __name__ == '__main__':

    image_folder = '' # Set the image folder path
    processor = SARDataProcessor(image_folder)
    A_amplitude = processor.average_amplitude(folder_path=image_folder, save_path=''save_path'')
    amplitude_nsd = processor.amplitude_nsd(folder_path=image_folder, save_path=''save_path'')